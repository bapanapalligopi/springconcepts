<!--Trivy scans using-->
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>exec-maven-plugin</artifactId>
				<version>3.1.0</version>
				<executions>
					<execution>
						<id>trivy-scan</id>
						<phase>verify</phase>
						<goals>
							<goal>exec</goal>
						</goals>
						<configuration>
							<executable>trivy</executable>
							<arguments>
								<!-- You can change fs to image or repo as
								needed -->
								<argument>fs</argument>
								<argument>.</argument>
								<!--  format for HTML report generation -->
								<argument>--format</argument>
								<argument>template</argument>
								<!-- Template file path -->
								<argument>--template</argument>
								<argument>@trivy-templates/html.tpl</argument> <!--
								Make sure this path is correct -->
								<!-- Output HTML report file -->
								<argument>--output</argument>
								<argument>
									target/qronposreconservices-trivy-report.html</argument>
								<!-- Specify scanners -->
								<argument>--scanners</argument>
								<argument>vuln,misconfig,secret</argument>
							</arguments>
						</configuration>
					</execution>
				</executions>
			</plugin>
