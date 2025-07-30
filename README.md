# Java, Maven & Tomcat Setup and Sample Deployment Guide

This README provides **step-by-step instructions** to set up Java OpenJDK 21, Apache Maven, and Apache Tomcat 10, and to create and deploy a sample Maven web application.

## 1. Install Java OpenJDK 21

### a. Detect your Linux distribution (Ubuntu/Debian or CentOS/RHEL/Rocky/AlmaLinux):

Open a terminal and run:

```bash
if [ -f /etc/os-release ]; then
  . /etc/os-release
  echo "Detected OS: $ID"
else
  echo "Unsupported OS."
  exit 1
fi
```

### b. Install Java and basic tools:

- **For Ubuntu/Debian:**

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk wget tar git
```

- **For CentOS/RHEL/Rocky/AlmaLinux:**

```bash
sudo dnf install -y java-21-openjdk-devel wget tar git
```

### c. Set `JAVA_HOME`:

```bash
JAVA_HOME_PATH=$(dirname $(dirname $(readlink -f $(which javac))))
echo "JAVA_HOME detected at: $JAVA_HOME_PATH"
```

### d. Persist Java settings (add to `~/.bash_profile`):

```bash
echo "export JAVA_HOME=$JAVA_HOME_PATH" >> ~/.bash_profile
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bash_profile
# For current session:
export JAVA_HOME=$JAVA_HOME_PATH
export PATH=$JAVA_HOME/bin:$PATH
```

### e. Test your Java installation:

```bash
java -version
```

## 2. Install Maven 3.9.11

### a. Download and extract Maven:

```bash
cd /tmp
wget https://downloads.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz
sudo mkdir -p /opt/maven
sudo tar xf apache-maven-3.9.11-bin.tar.gz -C /opt/maven --strip-components=1
```

### b. Set Maven environment:

```bash
echo 'export M2_HOME=/opt/maven' >> ~/.bash_profile
echo 'export PATH=$M2_HOME/bin:$PATH' >> ~/.bash_profile
export M2_HOME=/opt/maven
export PATH=$M2_HOME/bin:$PATH
```

### c. Verify Maven installation:

```bash
mvn -version
```

## 3. Install Apache Tomcat 10

### a. Download and extract Tomcat:

```bash
cd /tmp
wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.43/bin/apache-tomcat-10.1.43.tar.gz
sudo mkdir -p /opt/tomcat
sudo tar xf apache-tomcat-10.1.43.tar.gz -C /opt/tomcat --strip-components=1
```

### b. Set Tomcat environment:

```bash
echo 'export CATALINA_HOME=/opt/tomcat' >> ~/.bash_profile
echo 'export PATH=$CATALINA_HOME/bin:$PATH' >> ~/.bash_profile
export CATALINA_HOME=/opt/tomcat
export PATH=$CATALINA_HOME/bin:$PATH
```

### c. Make Tomcat scripts executable:

```bash
 sudo chmod 755 -R /opt/tomcat/
sudo chmod +x /opt/tomcat/bin/*.sh
```

### d. Start Tomcat:

```bash
sudo /opt/tomcat/bin/startup.sh
```

### e. Verify Tomcat installation:

Open a web browser and navigate to `http://localhost:8080`. You should see the Tomcat welcome page.

## 4. Create a Sample Maven Web Application

### a. Create a new Maven project:

```bash
mkdir -p ~/projects
cd ~/projects
mvn archetype:generate -DgroupId=com.example -DartifactId=sample-webapp -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
cd sample-webapp
```

### b. Update the `pom.xml` file:

Edit `pom.xml` to ensure it uses the correct Java version and dependencies:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <groupId>com.example</groupId>
  <artifactId>sample-webapp</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  
  <name>sample-webapp</name>
  <url>http://www.example.com</url>
  
  <properties>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  
  <dependencies>
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.0.0</version>
      <scope>provided</scope>
    </dependency>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  
  <build>
    <finalName>sample-webapp</finalName>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>21</source>
          <target>21</target>
        </configuration>
      </plugin>
      
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.3.2</version>
      </plugin>
    </plugins>
  </build>
</project>
```

### c. Build the web application:

```bash
mvn clean package
```

## 5. Deploy the Application to Tomcat

### a. Copy the WAR file to Tomcat's webapps directory:

```bash
sudo cp target/sample-webapp.war /opt/tomcat/webapps/
```

### b. Restart Tomcat:

```bash
sudo /opt/tomcat/bin/shutdown.sh
sudo /opt/tomcat/bin/startup.sh
```

### c. Test the deployment:

Open a web browser and navigate to `http://localhost:8080/sample-webapp`. You should see your web application.

## 6. Troubleshooting

### Common Issues:

- **Port 8080 already in use**: Check if another service is using port 8080 with `sudo netstat -tlnp | grep 8080`
- **Permission denied**: Ensure Tomcat scripts are executable with `sudo chmod +x /opt/tomcat/bin/*.sh`
- **JAVA_HOME not set**: Verify with `echo $JAVA_HOME` and ensure it's in your `~/.bash_profile`
- **Maven not found**: Check PATH with `echo $PATH` and verify Maven installation

### Logs:

- **Tomcat logs**: `/opt/tomcat/logs/catalina.out`
- **Application logs**: `/opt/tomcat/logs/localhost.YYYY-MM-DD.log`

## 7. Next Steps

- Configure Tomcat users in `/opt/tomcat/conf/tomcat-users.xml` for management access
- Set up SSL/TLS for production environments
- Configure database connections for your applications
- Implement CI/CD pipelines for automated deployment

---

**Note**: This guide assumes a Linux environment. For Windows or macOS, adjust paths and commands accordingly.
