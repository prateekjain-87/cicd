## CI/CD Usage
### Admin View
#### CI Pipeline
The CI pipeline has 3 parameters each having some default value as shown in the image below. The values of parameters can be set depending on the explanation below:

**Exit code**: The exit code parameter takes value for the exit code to be set for Trivy image scan. By default set to 0 and will let the normal pipeline build execution flow continue. If set to 1, the pipeline build will fail in case any specified severity level vulnerabilities are found in the docker image.

**Build Type**: The build type parameter takes the value of the type of build. The parameter can have values- Alpha, Beta, RC, and empty. By default, the value is empty which means it is a regular build.

**Build Number**: The build number necessarily donates the build number of a particular pipeline project. By default, an in-built Jenkins build number is used but it can be overridden using the current parameter.


The CI pipeline stage **'Build & Test'** has the logic written in the script section which will determine whether helm artifacts will be built and published to the artifactory. This stage also has an input parameter that will wait for user input on whether feature branch artifacts should be deployed or not. By default or if any input is not provided, it will be set to **'NO'**. The script code snippet is provided below for reference.

```sh
script{
    env.PUBLISH_ARTIFACTS = "true"
    env.PUBLISH_BUILDINFO = "true"
    if(env.GERRIT_EVENT_TYPE){
        if("${GERRIT_EVENT_TYPE}"!= "change-merged"){
            env.PUBLISH_ARTIFACTS = "false"
            env.PUBLISH_BUILDINFO = "false"
        }
    }
    if("${BRANCH_NAME}"!="develop" && "${env.PUBLISH_ARTIFACTS}" == "true")
    {
        try {
            timeout(time:420, unit:'SECONDS') {
                env.FEATURE_DEPLOY = input message: 'User input required',
                parameters: [choice(name: 'Deploy Feature branch artifacts?', choices: 'No\nYes', description: 'Choose "yes" if you want to deploy Feature branch artifacts')]
                BuildNumber = "${BRANCH_NAME}"
                BuildType = ""
            }
        } catch(err) {
                echo ("Input timeout expired, Setting default value as No")
                env.FEATURE_DEPLOY="No"
            }
    }
    if("${env.FEATURE_DEPLOY}"=="No")
    {
        env.PUBLISH_ARTIFACTS = "false"
        env.PUBLISH_BUILDINFO = "false"
    }
}
 ```

Further, based on the script evaluation, if publish artifacts is set to **'Yes'** then a maven profile **'generateDocker'** is executed which is responsible for the generation of Helm charts.

In rtMavenRun goal, multiple properties are being overridden onto the pom such as 'changelist', 'artifactory.publish.artifacts','artifactory.publish.buildInfo' and 'maven.test.skip' each deciding the behavior of build. The default values for each field are set and can be overridden. Below is a sample code snippet:

```sh
if (env.PUBLISH_ARTIFACTS == "true"){
    rtMavenRun(
        tool: "Maven 3.3.3",
        pom: '$WORKSPACE/pom.xml',
        goals: "clean install package -U -P generateDocker -Dchangelist=-${BuildType}${BuildNumber} -Dartifactory.publish.artifacts=${env.PUBLISH_ARTIFACTS} -Dmaven.test.skip=true -Dartifactory.publish.buildInfo=${env.PUBLISH_BUILDINFO}",
        resolverId: "MAVEN_RESOLVER",
        deployerId: "MAVEN_DEPLOYER",
        buildName: "${env.artifact}-${env.revision}",
        buildNumber: "${BuildType}${BuildNumber}"
    )
}
 ```

The stage **'Publish Helm Artifacts'** is responsible for pushing docker image and helm charts to Jfrog Artifactory. Further, it will also reindex the helm repository to update the index with newly pushed helm charts.

The stage **'Trivy Vulnerability Scan'** is responsible for scanning the docker images for any known vulnerabilities based on the command options set while executing the docker image scan run with trivy image. The behavior of the scan will be based on the options used and the values set for them. 

The report is visible of Jenkins pipeline build UI and can also be retrieved from the backend of Jenkins master server from the directory configured report directory.

The post action section is responsible for two tasks:

- To send email notifications on a failed build to the concerned developers whose commit caused the build to fail.

- To prepare release notes and publish them on the confluence page.

This section is configured based on the standard requirements and can be reconfigured if any changes or improvements are needed.

#### CD Pipeline
The CD pipeline has 4 parameters. Each parameter depicts the build version of services to be used for deployment and has some default value set with some prior known stable versions. The values of parameters can be set as the versions that the user wants to deploy.

The CD pipeline stages are effectively self-sufficient to carry out the whole deployment based on the environment-specific values provided in the env.properties file which is packaged in the same repository with the CD pipeline code. The file acts as an input to helm for required fields by setting the values in the form of environment variables. 

Essentially CD Pipeline has the following stages:

- env-setup

- Pull Artifacts

- Push Artifacts to AWS

- Deploy the artifacts to Dev Cluster 

- Deploy the artifacts to QA Cluster

The stage Deploy the artifacts to QA Cluster is an optional stage that requires the user to provide input on whether to do the deployment to QA Cluster as well. If no input is provided, the stage will be on hold and wait till any input is provided. Thus, the deployment can be done without any limit on the waiting time.

```sh
script {
    env.DEPLOY_ON_QA_ENV = input message: 'User input required',
    parameters: [choice(name: 'Deploy on QA Environment', choices: 'No\nYes', description: 'Choose "yes" if you want to deploy on QA Env')]
}
```

### User View
#### CI Pipeline
The CI pipeline is designed to trigger different Gerrit events and perform actions depending on the kind of Gerrit trigger event and the type of branch(develop/feature). The kind of event along with the type of branch decides whether to build and push the helm artifacts to Artifactory or not.

If a user pushes the code to any feature branch, the following events can get CI Pipeline triggered: Ref Updated, Draft Published, Patchset created, and Change Merged. Ref Updated event occurs mostly on all the pushes which leads to Ref change. Draft Published event occurs whenever a code is pushed for merging. Patchset event occurs whenever any code commits are amended. Change merged event occurs whenever the code changes are merged into the branch.

If a user merges a branch to develop a branch and pushes the changes to remote, a Ref Updated event occurs which triggers the CI pipeline with Publish Artifacts set to true. 

Whenever a code change is merged in any of the branches, again a pipeline build is triggered which essentially builds the branch with merged code.

This build will be by default eligible for helm artifacts build and push in case of develop branch but in case of a feature, branch user needs to provide input manually whether to publish the artifacts. By default, feature branch artifacts are not pushed to the Artifactory.

Further, the user can browse to the Jenkins Build UI to access SonarQube and Trivy vulnerability scan reports as highlighted in the screenshot below.


 

In case of any build failures, the user will get a mail from Jenkins with the build details. However, in case of build success, no mail will be sent.

In case the Jenkins build was on the 'develop' branch, the changelog will be updated on the Confluence with the build details. The changelog can be accessed using the Changelog link here.

 

#### CD Pipeline
The CD pipeline would require the user to update the env.properties file to provide environment-specific values in order to install helm charts with custom values for the fields required during the deployment of services. Further, the user needs to provide the build versions of the service to be deployed. Once done, the user can simply trigger the pipeline build.


In case, the user wants to deploy the services to the QA environment as well, an interactive input needs to be provided with choices as ‘Yes' or ‘No' to deploy the artifacts on the QA environment as well. The pipeline will be in a 'waiting’ state till any input is provided.


## Developer Guide
 

### POM Configuration
The changes required in the pom.xml file in order to run the CI pipeline with features of code coverage and test reports are the inclusion of the Maven Surefire and JaCoCo plugins in pom.xml. Further, a maven profile also needs to be created which will be run only when helm artifacts are desired to be produced.

The Surefire Plugin is used during the test phase of the build lifecycle to execute the unit tests of an application. It generates reports in two different file formats:

- Plain text files (*.txt)

- XML files (*.xml)

By default, these files are generated in ${basedir}/target/surefire-reports/TEST-*.xml.

The configuration required for Surefire plugin is as follows:

```sh
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>${surefire.version}</version>
            <configuration>
                <redirectTestOutputToFile>true</redirectTestOutputToFile>
            </configuration>
        </plugin>
    </plugins>
</build>

<reporting>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>${surefire.version}</version>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-report-plugin</artifactId>
            <version>${surefire.version}</version>
        </plugin>
    </plugins>
</reporting>
 ```

The JaCoCo Maven plug-in provides the JaCoCo runtime agent to our tests and allows basic report creation.

The JaCoCo Maven plug-in requires

- Maven 3.0 or higher and

- Java 1.5 or higher (for both, the Maven runtime and the test executor).

The below XML configuration:

- Prepares a property pointing to the JaCoCo runtime agent that can be passed as a VM argument to the application under test. 

- Creates a code coverage report for tests of a single project in multiple formats (HTML, XML, and CSV).

- Checks that the code coverage metrics are being met.

```sh
<build>
    <plugins>      
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>${jacoco.version}</version>
            <executions>
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                    <configuration>
                        <destFile>${basedir}/target/coverage-reports/jacoco-unit.exec</destFile>
                    </configuration>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                    <configuration>
                        <dataFile>${basedir}/target/coverage-reports/jacoco-unit.exec</dataFile>
                        <outputDirectory>${basedir}/target/jacoco-aggregate-ut/jacoco.xml</outputDirectory>
                    </configuration>
                </execution>
                <execution>
                    <id>check</id>
                    <goals>
                        <goal>check</goal>
                    </goals>
                    <configuration>
                        <title>Code Coverage of Unit Tests</title>
                        <!--<outputDirectory>target/jacoco-aggregate-ut</outputDirectory>-->
                        <rules>
                            <rule>
                                <element>CLASS</element>
                                <limits>
                                    <limit>
                                        <counter>LINE</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>${coverage}</minimum>
                                    </limit>
                                </limits>
                            </rule>
                        </rules>
                        <dataFile>${basedir}/target/coverage-reports/jacoco-unit.exec</dataFile>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
<build>
 ```

The maven profile required for generating helm artifacts is as follows:

```sh
<profile>
    <id>generateDocker</id>
    <build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>${maven.dockerfile.plugin.version}</version>
                <configuration>
                    <pullNewerImage>false</pullNewerImage>
                    <googleContainerRegistryEnabled>false</googleContainerRegistryEnabled>
                    <repository>${docker.registry}/${repo.path}/${project.artifactId}</repository>
                    <tag>${project.version}</tag>
                    <contextDirectory>${basedir}/src/main/docker</contextDirectory>
                </configuration>
                <executions>
                    <execution>
                        <id>default</id>
                        <goals>
                            <goal>build</goal>
                        </goals>
                        <phase>package</phase>
                    </execution>
                </executions>
            </plugin>					
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>${maven.assembly.plugin.version}</version>
                <configuration>
                    <appendAssemblyId>false</appendAssemblyId>
                    <descriptors>
                        <descriptor>src/assembly/assembly-helm.xml</descriptor>
                    </descriptors>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</profile>
 ```

## Jenkins Plugins Used
 

The CI Pipeline requires multiple plug-ins to be installed and configured in order to successfully run the multi-stage CI process. Namely, the following are the plugins needed:

**HTML Publisher**: Required for Publishing Trivy scan reports

**AWS Credentials**: Required for storing AWS Credentials for CLI interaction with AWS

**Git Changelog**: Required for fetching and formatting the Changelog between git commits 

**Confluence Publisher**: Required for publishing the changelog to Confluence page

**Build Timestamp Plugin**: Required to fetch the timestamp to include in the changelog

**Email Extension**: Required to make the emailing template more customizable

**Mailer**: Required to configure email notifications for build results.

**SonarQube Scanner**: Required to configure SonarQube server connection details in Jenkins global configuration.

The tools required to be installed on the build server includes:

- AWS CLI v2 with helm-s3 plugin
- Docker
- Helm v3
- Kubectl