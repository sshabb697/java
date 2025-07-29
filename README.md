

# Java, Maven \& Tomcat Setup and Sample Deployment Guide

This README provides **step-by-step instructions** to set up Java OpenJDK 21, Apache Maven, and Apache Tomcat 10, and to create and deploy a sample Maven web application.

## 1. Install Java OpenJDK 21

**a. Detect your Linux distribution (Ubuntu/Debian or CentOS/RHEL/Rocky/AlmaLinux):**

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

**b. Install Java and basic tools:**

- **For Ubuntu/Debian:**

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk wget tar git
```

- **For CentOS/RHEL/Rocky/AlmaLinux:**

```bash
sudo dnf install -y java-21-openjdk-devel wget tar git
```


**c. Set `JAVA_HOME`:**

```bash
JAVA_HOME_PATH=$(dirname $(dirname $(readlink -f $(which javac))))
echo "JAVA_HOME detected at: $JAVA_HOME_PATH"
```

**d. Persist Java settings (add to `~/.bash_profile`):**

```bash
echo "export JAVA_HOME=$JAVA_HOME_PATH" >> ~/.bash_profile
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bash_profile
# For current session:
export JAVA_HOME=$JAVA_HOME_PATH
export PATH=$JAVA_HOME/bin:$PATH
```

**e. Test your Java installation:**

```bash
java -version
```


## 2. Install Maven 3.9.11

**a. Download and extract Maven:**

```bash
cd /tmp
wget https://downloads.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz
sudo mkdir -p /opt/maven
sudo tar xf apache-maven-3.9.11-bin.tar.gz -C /opt/maven --strip-components=1
```

**b. Set Maven environment:**

```bash
echo 'export M2_HOME=/opt/maven' >> ~/.bash_profile
echo 'export PATH=$M2_HOME/bin:$PATH' >> ~/.bash_profile
export M2_HOME=/opt/maven
export PATH=$M2_HOME/bin:$PATH
```

**c. Verify Maven installation:**

```bash
mvn --version
```


## 3. Install Apache Tomcat 10.1.43

**a. Create tomcat system user (if doesn't exist):**

```bash
if ! id -u tomcat >/dev/null 2>&1; then
  sudo useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
else
  echo "User 'tomcat' already exists."
fi
```

**b. Download and install Tomcat:**

```bash
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.43/bin/apache-tomcat-10.1.43.tar.gz
sudo mkdir -p /opt/tomcat
sudo tar xf apache-tomcat-10.1.43.tar.gz -C /opt/tomcat --strip-components=1
```

**c. Set permissions:**

```bash
sudo chown -R tomcat: /opt/tomcat
sudo chmod +x /opt/tomcat/bin/*.sh
```

**d. Create systemd service for Tomcat:**

```bash
sudo tee /etc/systemd/system/tomcat.service > /dev/null <<EOF
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=${JAVA_HOME_PATH}"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
sleep 5
sudo systemctl status tomcat --no-pager
```


## 4. Create and Deploy a Sample Maven Web Application

### a. Generate Maven webapp project

```bash
mkdir -p ~/maven-tomcat-demo
cd ~/maven-tomcat-demo
mvn archetype:generate \
 -DgroupId=com.example \
 -DartifactId=mywebapp \
 -DarchetypeArtifactId=maven-archetype-webapp \
 -DinteractiveMode=false
cd mywebapp
```


### b. Replace the default `pom.xml` with the following:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>mywebapp</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <build>
    <finalName>${project.artifactId}##${project.version}</finalName>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
          <url>http://localhost:8080/manager/text</url>
          <username>admin</username>
          <password>adminadmin</password>
          <path>/${project.artifactId}##${project.version}</path>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```


### c. Add a simple `index.jsp`:

```bash
mkdir -p src/main/webapp
cat > src/main/webapp/index.jsp <<EOL
<html>
<body>
<h2>Hello World from Maven Webapp on Tomcat!</h2>
</body>
</html>
EOL
```


## 5. Configure Tomcat for Deployment

**Edit `/opt/tomcat/conf/tomcat-users.xml` and add these lines inside `<tomcat-users>`:**

```xml
<role rolename="manager-script"/>
<user username="admin" password="adminadmin" roles="manager-script"/>
```

**Restart Tomcat:**

```bash
sudo systemctl restart tomcat
```


## 6. Build and Deploy the Web Application

**a. Build the WAR file:**

```bash
mvn clean package
```

**b. Deploy the app to Tomcat:**

```bash
mvn tomcat7:deploy
```


## 7. Access Your Application

**Open the following URL in your browser:**

```
http://localhost:8080/mywebapp##1.0-SNAPSHOT
```

You should see:
**Hello World from Maven Webapp on Tomcat!**

## Troubleshooting

- **Java/Maven not found?**
Double-check the exported paths and restart your terminal session.
- **Tomcat not starting?**
Check `sudo systemctl status tomcat` for errors.
- **Deployment fails?**
Ensure the `admin` user with `manager-script` role is configured and Tomcat is restarted.


## Notes

- If you make changes to `pom.xml` or the source, **rebuild** with `mvn clean package` and **redeploy**.
- Remember to secure your Tomcat server for production environments.

Enjoy learning Java web development!

