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

How it works?
-------------

All JaCoCo magic is done in [`q7tests/pom.xml`][pom], lets learn it step by step.

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