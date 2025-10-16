# Deploying-a-Java-Web-App-with-Maven-Nexus-and-Tomcat

### Objective

The primary goal is to establish a robust, multi-server environment to build a Java web application, store the resulting artifact in a repository, and deploy it to a live web server. This guide details a complete CI/CD workflow using Maven for building, Nexus as an artifact repository, and Apache Tomcat for deployment, all hosted on AWS EC2 instances.


### Environment & Prerequisites

This setup involves two distinct servers to separate concerns and mimic a real-world production environment.

Build Server: An AWS EC2 instance dedicated to compiling the Java source code, running tests, and publishing the final deployable artifact (.war file) to Nexus.


Deployment Server: A second AWS EC2 instance that hosts the Apache Tomcat web server. Its role is to pull the application artifact and run it, making it accessible to users.


Tools: This guide uses Java 17, Maven 3.8.7, Nexus 3.85.0-03, and Apache Tomcat 9.

## 1. AWS EC2 Setup (Brief) ☁️

Create Two EC2 Instances

Build Server → For Java, Maven, and Nexus
Deploy Server → For Java and Tomcat


Choose AMI:
Ubuntu Server 24.04 LTS (64-bit x86)

<img width="953" height="747" alt="Image" src="https://github.com/user-attachments/assets/f7d9cd42-9d34-420a-a0f6-632fb72adcc1" />

Instance Type:
t2.micro (free tier)

<img width="959" height="197" alt="Image" src="https://github.com/user-attachments/assets/f7d87d0d-f3ed-466d-9e83-ea57f1173773" />

Ctreate a key pare

<img width="947" height="162" alt="Image" src="https://github.com/user-attachments/assets/1a9bddf1-c028-467e-9acd-d891677d1387" />

Security Groups:
Build Server → open ports 22, 8081, and 8080
Deploy Server → open ports 22 and 8080

<img width="563" height="785" alt="Image" src="https://github.com/user-attachments/assets/a952e021-62d5-421c-ad9f-71c3823e72fe" />

Key Pair:
Create and download a .pem key file (e.g., aws-key.pem)

<img width="947" height="162" alt="Image" src="https://github.com/user-attachments/assets/2271ab8f-7e33-4d3c-93ad-178424599a32" />

Connect to Instances:

```
chmod 400 aws-key.pem
ssh -i "aws-key.pem" ubuntu@<public-ip>
```

## 2. Build Server Setup ⚙️

Task Overview: The Build Server is the starting point of our pipeline. It is responsible for fetching the source code, compiling it into a deployable artifact using Maven, and then pushing that artifact to our Nexus repository for versioning and storage.

Steps:

### A. Install Java 17 and Maven

First, we need to install the Java Development Kit (JDK) and Maven, which are essential for building the application.

```
# 1. Update the server's package index
sudo apt update

# 2. Install OpenJDK version 17
sudo apt install openjdk-17-jdk -y

# 3. Verify the installation of Java runtime and compiler
java -version 

# 4. Download Maven
sudo apt -y install maven 

# 5. Verify the Maven installation
mvn -v

```

<img width="917" height="246" alt="Image" src="https://github.com/user-attachments/assets/b7a51720-3eb7-479a-a371-995166679b59" />

### B. Install Nexus Repository Manager

Next, we'll install Nexus to store our built artifacts.

```
# 1. Download the latest Nexus 3 Unix archive
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz -O nexus-3.85.0-03-linux-x86_64.tar.gz

# 2. Extract Nexus to the /opt directory
sudo tar -xzf nexus-3.85.0-03-linux-x86_64.tar.gz -C /

# 6. Reload systemd, then enable and start the Nexus service
sudo systemctl daemon-reload 
sudo systemctl enable nexus 
sudo systemctl start nexus 
sudo systemctl status nexus 

```

<img width="718" height="167" alt="Image" src="https://github.com/user-attachments/assets/b7ca7f6b-0aa3-4efa-8c0e-009922df0be7" />


if status check is nopt working use Net-Tools to see if nexus is running in its port

<img width="828" height="303" alt="Image" src="https://github.com/user-attachments/assets/8ba60934-e915-4c1d-bad0-d5e2bbbec9aa" />

### Deliverables:

✅ Java 17 and Maven 3.8.7 are successfully installed and configured on the Build Server.

✅ Nexus Repository Manager is running as a service and accessible via its web interface at http://<build-server-ip>:8081.


## 2. Configuring and Build surver 🛠️
Task Overview: With the tools in place, we must configure Maven to communicate with Nexus. We'll provide credentials for authentication and update the project's pom.xml file to define where the built artifact should be deployed.

### Steps:

A. Configure Maven and pom.xml

1.Configure Maven settings.xml: Edit the ~/.m2/settings.xml file on the Build Server. This step is crucial for authenticating with Nexus. The id in this file must match the repository id in the pom.xml.

<server>
    <id>releases</id> 
    <username>admin</username> 
    <password>YOUR_ADMIN_PASSWORD</password> 
</server>

<img width="523" height="240" alt="Image" src="https://github.com/user-attachments/assets/751de309-3466-4c30-9468-0eeac110ac25" />

2.Configure Project pom.xml: In your Java project's pom.xml file, add the <distributionManagement> section. This tells Maven the URL of the Nexus repository where the artifact will be sent upon deployment.

<distributionManagement>
    <repository>
        <id>releases</id> [cite: 56]
        <url>http://<build-server-ip>:8081/repository/releases/</url> [cite: 57]
    </repository>
</distributionManagement>

<img width="831" height="199" alt="Image" src="https://github.com/user-attachments/assets/e4645d50-46c8-49b0-8d48-f28e9d729d16" />

B. Clone and Prepare the Code
Before building the application, clone the project repository from GitHub and unzip it on the build server.

```
# 1. Navigate to home directory
cd ~/

# 2. Clone the repository
git clone https://github.com/<your-repo>/JavaWebCal.git

# 3. Navigate into the project directory
cd JavaWebCal

# 4. If your code is provided as a ZIP file, unzip it
unzip JavaWebCal.zip -d JavaWebCal
Once the code is in place, you can start building and deploying.
```

<img width="1057" height="234" alt="Image" src="https://github.com/user-attachments/assets/08efdf53-6ae0-40a4-90c8-d50e516d6c8b" />


C. Build and Publish the Artifact

Now, you can build the application and deploy the artifact to Nexus. Repeat these steps for each new version (e.g., 0.1, 0.2, 0.3).

```

# 1. Navigate to your project's root directory
cd ~/JavaWebCal [cite: 60]

# 2. Run 'mvn clean install'. This cleans previous builds and packages the application.
mvn clean install [cite: 61]

# 3. Run 'mvn deploy'. This uses the configuration from pom.xml and settings.xml
#    to upload the packaged artifact (.war file) to the Nexus repository.
mvn deploy [cite: 62]
```

<img width="1713" height="302" alt="Image" src="https://github.com/user-attachments/assets/64907157-2f5c-472a-8445-d14bc530e6d8" />

<img width="969" height="313" alt="Image" src="https://github.com/user-attachments/assets/9c2329a2-bb2d-4f46-b232-ff51e9d5daf0" />


Of course! Here is a more descriptive and detailed guide based on your provided document, using the structure and descriptive style of the Node.js task as a reference.

A Descriptive Guide to Deploying a Java Web App with Maven, Nexus, and Tomcat
Objective

The primary goal is to establish a robust, multi-server environment to build a Java web application, store the resulting artifact in a repository, and deploy it to a live web server. This guide details a complete  workflow using Maven for building, Nexus as an artifact repository, and Apache Tomcat for deployment, all hosted on AWS EC2 instances.



✅ Java 17 and Maven 3.8.7 are successfully installed and configured on the Build Server.

✅ Nexus Repository Manager is running as a service and accessible via its web interface at http://<build-server-ip>:8081.

<img width="588" height="525" alt="Image" src="https://github.com/user-attachments/assets/b00c7b7e-270c-44f3-890b-93f824d4459c" />

<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/943d9e93-6f34-4020-8e98-63f3c79cffdb" />

<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/ee3133e7-8e4f-4021-9a0c-91cb085268c7" />


<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/a687234f-c4f6-40c9-b257-d588edb69f07" />



## 3.Deployment Server Setup and Deployment �

�
The Deployment Server hosts the live web application.

It requires Java to run and Apache Tomcat as the web server.

Instead of manually copying the .war file, we’ll download itdirectly from the Nexus Repository.

```
A. Install Java and Tomcat
# 1. Update the package list
sudo apt update

# 2. Install OpenJDK 17 (required for the web app)
sudo apt install openjdk-17-jdk -y
java -version

# 3. Download Apache Tomcat 9
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.111/bin/apache-tomcat-9.0.111.zip

# 4. Extract Tomcat
unzip apache-tomcat-9.0.111.zip

# 5. Make startup/shutdown scripts executable
cd apache-tomcat-9.0.111/bin
chmod +x *.sh
```

<img width="1800" height="765" alt="Image" src="https://github.com/user-attachments/assets/f4ff4b0e-9220-4659-bdc9-d0bd3897e180" />


B. Retrieve the WAR File from Nexus
We’ll pull the latest .war artifact directly from the Nexus Repository Manager hosted on the build server.

```
# 1. Navigate to your home directory
cd ~

# 2. Download the WAR file directly from Nexus
# Replace <build-server-ip> and version accordingly
wget http://<build-server-ip>:8081/repository/releases/com/web/cal/webapp/0.2/webapp-0.2.war

# 3. Move the WAR file into Tomcat’s webapps directory
mv webapp-0.2.war /home/ubuntu/apache-tomcat-9.0.111/webapps/
```

<img width="1752" height="300" alt="Image" src="https://github.com/user-attachments/assets/f2874c2c-b526-49ab-ac07-cce9694656ef" />

C. Start Tomcat Server

```
# 1. Navigate to Tomcat’s bin directory
cd /home/ubuntu/apache-tomcat-9.0.111/bin

# 2. Start the Tomcat service
./startup.sh
```

Once started, Tomcat will automatically detect and deploy the webapp-0.2.war file.


### ✅ Deliverables

Java 17 and Tomcat 9 are installed on the deployment server.
WAR file downloaded directly from Nexus Repository.
Application successfully deployed and accessible via:
http://<deployment-server-ip>:8080/webapp-0.2/



## 3. End-to-End Verification ✅

To ensure the everything is working, perform a final check:

Trigger Build: Make a small change to your application code, update the version in pom.xml, and run mvn clean install deploy on the Build Server.

Verify in Nexus: Log in to the Nexus UI and confirm that the new version of your artifact has been uploaded successfully.

Deploy and Access: scp the new artifact to the Deployment Server, place it in the webapps directory, and restart Tomcat if necessary. Access the updated application in your browser to verify the changes are live.

Conclusion & Summary

This exercise demonstrates a fundamental DevOps workflow. By separating the build and deployment environments and using a central artifact repository, you create a scalable, reliable process for deploying a Java web application.


<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/956e3a8c-b531-483b-97b0-7bd9f6b2b1e9" />


<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/98d1b3fd-e0de-4b5c-8b2b-742185f2277e" />

