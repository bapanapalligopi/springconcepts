```java
<!-- JaCoCo Plugin for Code Coverage -->
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>${jacoco-maven-plugin-version}</version>
				<executions>
					<!-- Prepares the JaCoCo agent -->
					<execution>
						<id>prepare-agent</id>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution>
					<!-- Generates coverage report after tests -->
					<execution>
						<id>report</id>
						<phase>verify</phase> <!-- changed to `verify` for
						cleaner lifecycle -->
						<goals>
							<goal>report</goal>
						</goals>
					</execution>
					<!-- Optional: Console report during test phase -->
					<execution>
						<id>console-report</id>
						<phase>test</phase>
						<goals>
							<goal>report</goal>
						</goals>
						<configuration>
							<outputDirectory>
								${project.build.directory}/qronposreconservices-jacoco-report</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
```
