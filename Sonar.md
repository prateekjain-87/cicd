## Setting up Sonarqube container
 

In order to integrate SonarQube with Jenkins server, we will require to either run SonarQube on Docker or install on local system using packages.

We will run SonarQube on Docker for our dev/test purposes.

 

For Running SonarQube on Docker, execute the following command:
```sh
$ docker run -d --name sonarqube -v sonarqube_data:/opt/sonarqube/data -v sonarqube_extensions:/opt/sonarqube/extensions -v sonarqube_logs:/opt/sonarqube/logs   -p 9000:9000 docker.io/sonarqube:8.9.6-community
```
After the container is up and running, access the Sonar UI using the following command: 
```sh
$ http://{hostname}:9000
```
 

Note that we might need to wait a few seconds while SonarQube is starting to see the screen. Login to SonarQube with the default admin user and admin password.

### SonarQube Scanner Configuration
 
SonarQube Scanner (aka Sonar Scanner) is a stand alone tool that does the actual scanning of the source code and sends results to the SonarQube Server. In our simple setup, we will install Sonar Scanner on the same server/container as Jenkins, but in a production environment it would most likely be on a separate machine/container/VM. 

**Steps for sonar-scanner installation**

- Install SonarQube Scanner Jenkins plugin (Manage Jenkins > Manage Plugins > Available)

- Access the Jenkins server/container from a bash shell

- Create sonar-scanner directory under /var/jenkins_home

- Download SonarQube Scanner onto the container from the sonar-scanner directory with wget:
```sh
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
```
- Unzip the Sonar Scanner binary:
```sh
unzip sonar-scanner-cli-3.3.0.1492-linux.zip
```
- Update Jenkins to point to sonar-scanner binary (Manage Jenkins > Global Tool Configuration > SonarQube Scanner); we will need to uncheck “Install automatically” so we can explicitly set SONAR_RUNNER_HOME

**Note:** We can also configure Sonar Scanner using “Install automatically” option also (Manage Jenkins > Global Tool Configuration > SonarQube Scanner)


### Configuring Jenkins and SonarQube
 
After Jenkins and SonarQube are running, we will configure them to communicate with each other.

**Scenario 1: Using hostname**

Add webhook in SonarQube to point to Jenkins (Administration > Configuration > Webhooks); URL will be in the format http://{hostname}:{port}/sonarqube-webhook  (http://myserver:8081/sonarqube-webhook)

**Scenario 2: Using IP Address**

Get IP address of host by executing from the host:
```sh
$ ifconfig
```
Look for the IP address listed for en0 > inet:

Add webhook in SonarQube to point to Jenkins (Administration > Configuration > Webhooks); URL will be in the format http://<host_ip>:{port}/sonarqube-webhook  (http://192.168.0.13:8080/sonarqube-webhook)

**Note:** If we change networks, we will have to update the IP address on Jenkins and SonarQube to the new host IP.

 

After configuring webhook, perform the following steps:

In SonarQube, generate an access token that will be used by Jenkins (My Account > Security > Tokens)

After the access token is generated, add a secret in Jenkins so that it can be referred while configuring Sonarqube in Jenkins.

In Jenkins, add the SonarQube Server IP address/hostname and the access token (Manage Jenkins > Configure System > SonarQube Servers); URL will be in the format http://<host_ip>:9000   (http://192.168.0.13:9000) or http://{hostname}:9000 (http://myserver:9000)

 

### Conclusion
We now have SonarQube and Jenkins configured to work together. We can now create Jenkins pipeline jobs to start analyzing our projects.