```java
<!--Html Vulnerability Report usind OWASP-->
			<plugin>
				<groupId>org.owasp</groupId>
				<artifactId>dependency-check-maven</artifactId>
				<version>12.1.0</version>
				<executions>
					<execution>
						<goals>
							<goal>check</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<!-- Set the report format to HTML -->
					<format>HTML</format>
					<!-- Specify an output directory for the report -->
					<outputDirectory>
						${project.build.directory}/qronposreconservices-dependency-check-report</outputDirectory>
				</configuration>
			</plugin>
```
