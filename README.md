# 🚀 Java, Maven, and Tomcat Deployment Lab

This lab provides a step‑by‑step guide to install **Java 21**, **Maven 3.9.11**, and **Apache Tomcat 10.1.43**, then build and deploy a sample **Maven web application**.

---

## 📌 Prerequisites
- A Linux system (Ubuntu, Debian, CentOS, Rocky, or AlmaLinux)
- `sudo` privileges
- Internet access

---

## 📝 Step 1: Detect Your Linux Distribution

```bash
cat /etc/os-release
Look for the value of ID (e.g., ubuntu, debian, centos, rocky, etc.).

This will determine which package manager we use later.

## ☕ Step 2: Install Java 21

For Ubuntu/Debian
```bash
   sudo apt update
sudo apt install -y openjdk-21-jdk wget tar git

For CentOS/RHEL/Rocky/AlmaLinux
```bash
sudo dnf install -y java-21-openjdk-devel wget tar git

✅ Verify installation:

```bash
java -version




