## Maven Build, Tomcat Deployment and Nexus Push

Deploying a simple web application on tomcat using apache maven, and pushing the artifact to a nexus repository.

### Tools and Packages

![aws](https://img.shields.io/badge/amazonaws-232F3E?style=for-the-badge&labelColor=black&logo=amazonaws&logoColor=white) ![amazon ec2](https://img.shields.io/badge/amazonec2-FF9900?style=for-the-badge&labelColor=black&logo=amazonec2&logoColor=FF9900) ![Maven](https://img.shields.io/badge/apachemaven-3-C71A36?style=for-the-badge&logo=apachemaven) ![Tomcat](https://img.shields.io/badge/apachetomcat-9-F8DC75?style=for-the-badge&logo=apachetomcat) ![Sonatype](https://img.shields.io/badge/sonatype-3-1B1C30?style=for-the-badge&logo=sonatype) ![javajdk](https://img.shields.io/badge/javajdk-17-437291?style=for-the-badge&logo=openjdk)


### Maven Server Setup and Configuration

1. Launch an Amazon Linux EC2 instance server on AWS

![amazonlinux pic or ec2 maven pic](https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush/blob/main/images/amazonlinux.png)

2. Install Java JDK 17

```yum install java-17 -y```

```java -version``` to confirm installation

3. After installation, we need to add the java path to the bash profile
   
```vi ~/.bash_profile```

4. Use ```find /usr/lib/jvm/java-17* | head -n3``` to find the path.

![jdk17 to path pic](https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush/blob/main/images/jdk17%20to%20path.png)

- To activate: ```source ~/.bash_profile```

- ```echo $PATH``` to confirm.

5. Next, we download maven into the /opt/ directory (the directory is just a choice)

- Download maven packages from https://maven.apache.org/download.cgi

```
wget https://dlcdn.apache.org/maven/maven-3/3.9.4/binaries/apache-maven-3.9.4-bin.tar.gz
```
- Extract using ```tar -xvzf apache-maven-3.9.4-bin.tar.gz```

6. cd into the folder and use pwd to get the working directory which we will add to the bash_profile

![path pic](https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush/blob/main/images/path%20pic.png)

7. Now maven is successfully installed
  
Use ```mvn -version``` to confirm installation.

### Obtaining the webapp for deployment

1. Install git and clone a simple webapp repository (https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush.git) into any suitable directory on the maven server.

- This directory is cloned from [seunaolu](https://github.com/seunayolu/maven-build-website), and the webapp is a template from [tooplate](https://www.tooplate.com/view/2136-kool-form-pack)
  
- Install tree ```yum install tree -y```
  
2. cd into the webapp directory and view the directory structure using ```tree```
  
![dir structure pic](https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush/blob/main/images/dir%20structure.png)

- ```vi pom.xml``` to view the contents of the pom.xml (this contains instructions for the application)

### Building the webapp

1. We can run any of the maven goals ```validate, compile, test, package, install, deploy```
  
- Run ```mvn clean``` first to remove any previously created 'target' directory.
  
2. ```mvn package``` (this creates a target folder with snapshot and a jar executable .war file)
  
- after running maven gaols, a **.m2 directory** is created in the ~ directory. This directory is where we add a settings.xml file 2ru which we'll provision access to maven artefacts.

### Tomcat Server Setup and Configuration

- Now, let's deploy a Tomcat server on AWS
![tomcat server or ec2 tomcat](pic link)
- By default, tomcat runs on port 8080. We'll, however, open tomcat up on a different port, say 8090. (In my environment, I already have Jenkins running on port 8080).
- Install java17 and provision environmental variables in .bash_profile (Just like before)
- Now, install tomcat in the /opt/ directory (just to be consistent)
- Download tomcat packages from https://tomcat.apache.org/download-90.cgi
```wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.80/bin/apache-tomcat-9.0.80.tar.gz```
- After extraction, cd into the directory.
- Next, cd into the /bin directory and give execute right to our users in the startup and shutdown scripts.
```chmod +x startup.sh```
```chmod +x shutdown.sh```
![chmod pic](pic link)
- Let's create a link file to the startup and shutdown scripts
```ln -s /opt/apache-tomcat-9.0.80/bin/startup.sh /usr/local/bin/tomcatup```
```ln -s /opt/apache-tomcat-9.0.80/bin/shutdown.sh /usr/local/bin/tomcatdown```
![start and stop pic](pic link)
- Now, from any directory, we can start and stop tomcat using ```tomcatup``` and ```tomcatdown``` respectively.
- Now let's change the default Tomcat port.
- We do that by editing the **/conf** directory ```server.xml``` in the tomcat directory installation.
![conf pic](pic link)
- Under the **connector** section, we edit 8080 to 8090.
![connector pic](pic link)
- We also need to open up port 8090 in our aws security group.

- Again, by default, tomcat does not allow login from a browser, so we need to fix that.
- We need to edit all the **context.xml** files in **webapps/** directory inside the tomcat directory installation.
```find / -name context.xml```
![contextxml pic](pic link)
- We need to ```vi``` into these four files
![vi-into pic](pic link)
- Inside these four files, we need to **comment out** the **<Valve** section.
![valve1 pic](pic link)
![valve2 pic](pic link)
![valve3 pic](pic link)
![valve4 pic](pic link)

- Now, we can start tomcat ```tomcatup```
- Then we can access the webserver on the browser using the ip and port specified
![tomcat browser pic](pic link)

- Inorder to access the "Manager App" as "Host Manager", we need to provide user details in **conf/tomcat-users.xml**
-  Now add the following rolename values to the file:
```bash
 <role rolename="manager-gui"/>
 <role rolename="manager-script"/>
 <role rolename="manager-jmx"/>
 <role rolename="manager-status"/>
 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
 <user username="deployer" password="deployer" roles="manager-script"/>
 <user username="tomcat" password="s3cret" roles="manager-gui, admin-gui"/>
 ```

- Now let's log into tomcat manager app using the "tomcat" username and password.
- We now have access to all sections of tomcat and we can now begin deploying our web-applications from maven to tomcat.
![manager pic](pic link)
![host manager pic](pic link)

### Deployment to Tomcat

- On the maven server, we can use ```mvn clean``` to clean our webapp.
- Let's open the pom.xml
    - Here, we specify packaging as 'war' instead of the default 'jar'
    - We also use the **tomcat7-maven-plugin** inorder to deploy directly to tomcat
- Inorder for maven to access tomcat, we need to create and configure a ```settings.xml``` file in **~.m2** as stated earlier
![m2 pic](pic link)
- Inside .m2 directory, do ```vi settings.xml```
- Then configure access with username and password
```bash
<?xml version="1.0" encoding='UTF-8"?>
<settings>
    <servers>
        <server>
            <id>Tomcat-Server</id>
            <username>admin</username>
            <password>admin</password>
        </server>
    </servers>
</settings>
```
![settingsxml pic](pic link)

- Now, let's deploy
```mvn tomcat7:deploy```
![tomcat success pic](pic link)

- Our deployment will be in the 'webapps' directory of the Tomcat server installation similar to the tomcat web application manager on the web
![deployment1 pic](pic link)
![deployment2 pic](pic link)

- Now, our webapp is up and running on the browser
![earthapp pic](pic link)


### Setting Up Nexus Repository

- Let's start up a Nexus server on aws (Nexus requires at least 4 vCPU to run but we'll nevertheless use t3-medium to conserve cost)
![nexusaws pic](pic link)
- Next, we'll open up port 8081 on the security group as nexus requires that.
- Nexus works with java 1.8, so install ```yum install java-1.8* -y``` and configure the paths in **.bash_profile**
- Now, let's download nexus ```wget https://download.sonatype.com/nexus/3/nexus-3.59.0-01-unix.tar.gz``` into /opt directory.
- After extraction, we get a **nexus...** directory and a **sonatype-work** directory
![nexusdir pic](pic link)
- cd into the **nexus...** directory and start nexus from the bin directory ```bin/nexus start```
- Nexus warns against starting as root user so let's create a new user for nexus and change ownership of those repositories to it.
![nexuswarn pic](pic link)
- Create a new user ```useradd nexus```
- Change the password ```passwd nexus```
- Change ownership of the two nexus directories to the new user
![chown pic](pic link)
- Next we need to specify the user to run nexus with in the **nexus.rc** file inside the **bin/** directory
![runasuser pic](pic link)
- Now, let's change user ```su -nexus``` and start nexus ```bin/nexus start```
![start nexus pic](pic link)
- Start nexus in browser using ip address and port
![signin pic](pic link)
- We need to get the password as specified by nexus and sign in with **admin** username
- Let's create a new blob in addition to the default. We'll call it **"maven-repo"**.
![blob pic](pic link)
- We can also see it reflect on the command line
![blobcmd pic](pic link)
- Next, lets create a new snapshot repository on the nexus browser console. We'll call it **"maven-app"**.
![snapshot pic](pic link)


### Pushing Artifact to Nexus Repository

- On the maven server, we need to edit the "pom.xml" with instructions for nexus push by adding a "distribution management" section
![distmgnt pic](pic link)
- We also need to edit the "settings.xml" file in .m2 directory
- But first, on the nexus repository web console, we need to create a **user** with priviledges to push into the repository. This priviledges are defined under created **roles.**
- So, we created a role called "maven-app" and attached "view*" priviledges to it.
![role1 pic](pic link)
![role2 pic](pic link)
- Then we created a user called "devops-user" and attached the maven-app role to it.
![devopsuser pic](pic link)
- Now, let's include these details in the settings.xml

```bash
<?xml version="1.0" encoding='UTF-8"?>
<settings>
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username>devops-user</username>
            <password>mypassword</password>
        </server>
    </servers>
</settings>
```
- Finally, let's clean, rebuild and deploy to nexus ```mvn clean deploy```
![pushsuccess pic](pic link)
![pushsuccess2 pic](pic link)
![pushsuccess3 pic](pic link)
- The artefact is also present on the server in the directory "sonatype-work/nexus3/blobs/maven-repo/content"
![pushsuccess4 pic](pic link)
