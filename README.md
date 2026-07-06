# Jenkins-Zero-To-Hero

Are you looking forward to learn Jenkins right from Zero(installation) to Hero(Build end to end pipelines)? then you are at the right place. 

## Installation on EC2 Instance

YouTube Video ->
https://www.youtube.com/watch?v=zZfhAXfBvVA&list=RDCMUCnnQ3ybuyFdzvgv2Ky5jnAA&index=1


![Screenshot 2023-02-01 at 5 46 14 PM](https://user-images.githubusercontent.com/43399466/216040281-6c8b89c3-8c22-4620-ad1c-8edd78eb31ae.png)

Install Jenkins, configure Docker as agent, set up cicd, deploy applications to k8s and much more.

## AWS EC2 Instance

- Go to AWS Console
- Instances(running)
- Launch instances

<img width="994" alt="Screenshot 2023-02-01 at 12 37 45 PM" src="https://user-images.githubusercontent.com/43399466/215974891-196abfe9-ace0-407b-abd2-adcffe218e3f.png">

### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins

Install Java

**Note:** Jenkins now requires Java 21+. Install Java 21:

```
sudo apt update
sudo apt install openjdk-21-jre -y
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins

**Note:** The `jenkins.io-2023.key` is outdated. Use the following commands to fetch the current signing key and install Jenkins.

```
sudo gpg --keyserver keyserver.ubuntu.com --recv-keys 7198F4B714ABFC68
sudo gpg --export 7198F4B714ABFC68 | sudo tee /usr/share/keyrings/jenkins-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.gpg] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y
```

**Troubleshooting:** If `apt-get install jenkins` fails due to an unsupported Ubuntu version (e.g. 24.10+), install Jenkins directly via the `.war` file instead:

```
wget https://get.jenkins.io/war-stable/latest/jenkins.war
java -jar jenkins.war --httpPort=8080
```

**Troubleshooting:** If Jenkins fails to start with `Running with Java 17... Supported Java versions are: [21, 25]`, force Java 21 by overriding the service:

```
sudo systemctl edit jenkins
```

Add the following and save:

```
[Service]
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64"
```

Then reload and start:

```
sudo systemctl daemon-reload
sudo systemctl start jenkins
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

### Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">

Jenkins Installation is Successful. You can now starting using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">

## Install the Docker Pipeline plugin in Jenkins:

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline".
   - Select the plugin and click the Install button.
   - Restart Jenkins after the plugin is installed.
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.


## Docker Slave Configuration

Run the below command to Install Docker

```
sudo apt update
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

## SonarQube Installation

**Pre-Requisites:**
- EC2 instance with at least 12GB disk and 4GB RAM
- Java 21 (already installed for Jenkins)
- A dedicated `sonarqube` user

### Setup

```
sudo apt update && sudo apt install unzip -y
sudo adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.6.0.92116.zip
sudo unzip sonarqube-10.6.0.92116.zip -d /opt/
sudo mv /opt/sonarqube-10.6.0.92116 /opt/sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
sudo chmod -R 775 /opt/sonarqube
```

### Start SonarQube

```
sudo su - sonarqube
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
./sonar.sh status
```

### Access SonarQube

Open port 9000 in your EC2 security group inbound rules, then navigate to:

```
http://<ec2-instance-public-ip>:9000
```

Default credentials: `admin` / `admin`

**Note:** Use SonarQube 10.6+ — earlier versions (e.g. 10.4.1) are incompatible with Java 21 due to removal of the Java Security Manager.

## Pipeline Troubleshooting

### Docker Agent: Java Version

The original pipeline used `abhishekf5/maven-abhishek-docker-agent:v1` which ships with Java 11. SonarQube 10.6 downloads a scanner bootstrap jar compiled with Java 17 (class file version 61), causing:

```
UnsupportedClassVersionError: class file version 61.0, this version only recognizes up to 55.0
```

The Jenkinsfile has been updated to use the official `maven:3.9-eclipse-temurin-17` image instead, which is multi-arch (amd64/arm64) and includes Java 17.

### Permission Denied on Checkout

The Docker agent runs as `root`, so build artifacts in `target/` are owned by root. On subsequent runs, the `jenkins` user cannot remove them during git checkout:

```
error: unable to unlink old '...target/spring-boot-web.jar': Permission denied
```

The Checkout stage now cleans `target/` at the start of each run:

```groovy
sh 'rm -rf java-maven-sonar-argocd-helm-k8s/spring-boot-app/target || true'
```

**Note:** Do not use `sudo` in pipeline steps — the Docker agent container does not have it installed.

