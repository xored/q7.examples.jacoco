q7.examples.jacoco
==================

Illustrates how to collect code coverage information during Q7 tests.

How to run?
-----------

To run the testing and coveraging you need [`mvn`][maven] on search path of your console. Run from console:

    git clone https://github.com/xored/q7.examples.jacoco
    cd q7.examples.jacoco
    mvn clean verify
    
[maven]: http://maven.apache.org/

All JaCoCo magic is done inside [`q7tests/pom.xml`][pom], lets learn it step by step.

Collecting coverage data
------------------------

[pom]: https://github.com/xored/q7.examples.jacoco/blob/master/q7tests/pom.xml

[JaCoCo][jacoco] has the [Maven plugin][maven-plugin], to add it to our Maven build, we need a Maven plugin repository to fetch it from. Unfortunately, JaCoCo snapshots repository is broken at the time this text was written, August 2012, so we will fetch an outdated version from [The Central Repository][central-repo]:

    <pluginRepository>
        <id>central</id>
        <name>Maven Central Repository</name>
        <url>http://repo1.maven.org/maven2/</url>
    </pluginRepository>

[jacoco]: http://www.eclemma.org/jacoco/
[maven-plugin]: http://www.eclemma.org/jacoco/trunk/doc/maven.html
[central-repo]: http://search.maven.org/#search|ga|1|g%3Aorg.jacoco

Maven build must be somehow instructed to activate coverage data collection, that is easy:

    <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.5.9.201207300726</version>
		<executions>
			<execution>
                <configuration>
                    <destFile>${project.basedir}/target/jacoco/data.exec</destFile>
                    <propertyName>jacocoArgLine</propertyName>
                </configuration>
				<goals>
					<goal>prepare-agent</goal>
				</goals>
			</execution>
		</executions>
    </plugin>

That specifies where coverage data will be stored using `destFile` and yields `jacocoArgLine` property AKA variable using `propertyName`, which will be plugged into [Q7 Maven Plugin][q7-plugin]:

    <vmArgs>
        <vmArg>${jacocoArgLine}</vmArg>
    </vmArgs>

[q7-plugin]: http://help.xored.com/display/Q7/Q7+Maven+Plugin

Generating coverage report
--------------------------

It is obvious to use [report goal][report-goal] from JaCoCo Maven plugin:

	<execution>
		<id>report</id>
		<phase>prepare-package</phase>
		<configuration>
			<dataFile>${project.basedir}/target/jacoco/data.exec</dataFile>
			<outputDirectory>${project.basedir}/target/jacoco/report</outputDirectory>
		</configuration>
		<goals>
			<goal>report</goal>
		</goals>
	</execution>

Bad news, it will not work as you wish in most cases. The goal searches for source code at some predefined location relative to `project.basedir` and have no options to configure the report, that is not so much handy in everyday life. We will work around this, let the black magic starts.

[report-goal]: http://www.eclemma.org/jacoco/trunk/doc/report-mojo.html

The Ant magic
-------------

Luckily enough, there are [JaCoCo Ant plugin][jacoco-ant] and [Maven AntRun plugin][maven-ant], using the second one we will be able to run Ant tasks from the first one:

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.7</version>
        <dependencies>
            <dependency>
                <groupId>org.jacoco</groupId>
                <artifactId>org.jacoco.ant</artifactId>
                <version>0.5.9.201207300726</version>
            </dependency>
            <dependency>
                <groupId>ant-contrib</groupId>
                <artifactId>ant-contrib</artifactId>
                <version>20020829</version>
            </dependency>
        </dependencies>
        <executions>
            <execution>
                <id>jacoco-report</id>
                <phase>prepare-package</phase>
                <goals>
                    <goal>run</goal>
                </goals>
                <configuration>
                    <target>
                        <property name="source-location" location="../org.eclipse.ui.examples.rcp.browser"/>
                        <taskdef name="jacoco-report" classname="org.jacoco.ant.ReportTask" classpathref="maven.plugin.classpath" />
                        <taskdef classpathref="maven.runtime.classpath" resource="net/sf/antcontrib/antcontrib.properties" />
                        <jacoco-report>
                            <executiondata>
                                <file file="${project.basedir}/target/jacoco/data.exec" />
                            </executiondata>

                            <structure name="org.eclipse.ui.examples.rcp.browser">
                                <classfiles>
                                    <fileset dir="${source-location}/target/browser.jar-classes" />
                                </classfiles>
                                <sourcefiles>
                                    <fileset dir="${source-location}/src" />
                                </sourcefiles>
                            </structure>
                            <html destdir="${project.basedir}/target/jacoco/report" />
                            <xml destfile="${project.basedir}/target/jacoco/report/jacoco.xml"/>
                        </jacoco-report>
                    </target>
                </configuration>
            </execution>
        </executions>
    </plugin>

Bingo! That way we may freely setup the report to include classes we wish using `structure` and its `classfiles` and `sourcefiles`. Even more, if you interested in coverage report for diferent areas of your code, you may setup `group`s:

	<structure name="Product Title">
	    <group name="Networking">
	        <classfiles>
	            <fileset dir="${source-location}/networking/bin" />
	        </classfiles>
	        <sourcefiles>
	            <fileset dir="${source-location}/networking/src" />
	        </sourcefiles>
	    </group>    
	    <group name="Rendering">
	        <classfiles>
	            <fileset dir="${source-location}/rendering/bin" />
	        </classfiles>
	        <sourcefiles>
	            <fileset dir="${source-location}/rendering/src" />
	        </sourcefiles>
	    </group>    
	</structure>

That's all, the puzzle is done, all pieces are aligned. Thanks to [David Carver][david] for his great "[Jacoco, Tycho, and Coverage Reports][article]".

[jacoco-ant]: http://www.eclemma.org/jacoco/trunk/doc/ant.html
[maven-ant]: http://maven.apache.org/plugins/maven-antrun-plugin/
[david]: http://intellectualcramps.wordpress.com/about/
[article]: http://intellectualcramps.wordpress.com/2012/03/22/jacoco-tycho-and-coverage-reports/