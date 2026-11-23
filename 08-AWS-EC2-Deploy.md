# Week 12 — Ubuntu VM on AWS EC2 & deploying a web app

**Goal:** push a Maven web app to GitHub → launch an Ubuntu EC2 instance → install Docker/Git → clone the app → build a Docker image → run it → open it via the EC2 **public IP**.

**EC2** = Amazon Elastic Compute Cloud (virtual machines in the cloud; no upfront hardware cost).

---

## Step I — Application on GitHub
Create the Maven web project in Eclipse and push it (see `03-Git.md` → section B).

## Step II — Launch the Ubuntu EC2 instance

1. Log in to AWS Console → **Services → EC2** → pick a nearby **Region** → **Launch instance**.
2. **Name:** `ubuntu`.
3. **AMI:** *Ubuntu Server* (**Free tier eligible**).
4. **Architecture:** 64-bit (x86).
5. **Instance type:** `t2.micro` (1 vCPU, 1 GB RAM — free tier).
6. **Key pair:** *Create new key pair* → download the `.pem` file → store in a folder (e.g. `AWS`).
7. **Network settings → Security group:** create new; tick **Allow HTTP** and **Allow HTTPS**; (SSH 22 is allowed by default).
8. **Storage:** 8 GB (root volume — equivalent to the C: drive).
9. **Number of instances:** 1 → **Launch instance**.
10. Select the instance → **Connect → SSH client** → copy the `ssh -i …` command.

## Step III — Connect and install software

From PowerShell / Git Bash, go to the folder containing the key, then paste the command from the console:
```bash
cd /path/to/AWS
chmod 400 mykey.pem                          # Linux/Mac/Git Bash
ssh -i "mykey.pem" ubuntu@<public-ip-or-dns>
```
On the instance:
```bash
sudo apt update
sudo apt-get install docker.io -y
sudo apt install git -y
sudo apt install nano -y
sudo systemctl enable --now docker
```
Clone the project:
```bash
git clone https://github.com/<user>/<maven-web-java-repo>.git
cd <maven-web-java-repo>
```

## Step IV — Write the Dockerfile
```bash
nano Dockerfile
```
Paste (builds the WAR inside Docker, then runs it on Tomcat — works even if `target/` isn't committed):
```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

FROM tomcat:9.0-jdk17
COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
```
Save in nano: **Ctrl+O**, **Enter**, **Ctrl+X**.

Simpler version if the `.war` is already in the repo (e.g. `target/mywebapp.war`):
```dockerfile
FROM tomcat:9.0
COPY target/mywebapp.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
```

## Step V — Build, run, access
```bash
sudo docker build -t webapp-image .
sudo docker images
sudo docker run -d --name app-demo -p 6060:8080 webapp-image
sudo docker ps
```
Open in browser: `http://<EC2-public-IP>:6060/`

**Cannot connect?** Edit the **Security Group → Inbound rules → Add rule:** Custom TCP, port **6060**, Source `0.0.0.0/0` (or "Anywhere-IPv4"). Also check `sudo docker logs app-demo`.

**Clean up:** `sudo docker stop app-demo && sudo docker rm app-demo && sudo docker rmi webapp-image`, then **terminate the instance** from the EC2 console to avoid charges.

> Tip: t2.micro has only 1 GB RAM; if the in-Docker Maven build is slow or gets killed, build the WAR locally with `mvn clean package`, commit/copy it, and use the simpler Dockerfile.

---

## Scenario answers

1. **index.html not loading on a new Amazon Linux 2 EC2:** connect via SSH → `sudo yum install httpd -y` → `sudo systemctl start httpd` → copy `index.html` to `/var/www/html/` → allow **HTTP (port 80)** in the Security Group → open `http://<public-ip>`. (On Ubuntu: `sudo apt install apache2`, files in `/var/www/html/`, service `apache2`.)
2. **Deployed to /var/www/html but still not loading:** (a) check Apache is running — `sudo systemctl status httpd`; (b) check **port 80 allowed** in the Security Group.
3. **Apache stops after reboot:** `sudo systemctl enable httpd` (auto-start at boot).
4. **Install Maven on EC2:** Amazon Linux: `sudo yum install maven -y`; Ubuntu: `sudo apt install maven -y`; verify `mvn -version`.
5. **Where to put the WAR for Tomcat:** `<TOMCAT_HOME>/webapps/` — Tomcat auto-deploys WARs placed there.
6. **404 from Tomcat app:** check the WAR is in `webapps/` and got exploded, correct **context path** in the URL (`/<warname>/`), Tomcat is running, and read the Tomcat logs (`logs/catalina.out`).
7. **Works with `curl localhost:8080` on EC2 but not from browser:** the Security Group has no **inbound rule for TCP 8080** — add it.
8. **Public IP changes:** use an **Elastic IP** (static public IPv4 address).
9. **Auto-run commands at instance setup/launch:** **EC2 User Data** (startup script).
10. **Build needs dependencies but no internet:** place the instance in a private subnet with a **NAT Gateway** (in a public subnet) for secure outbound-only internet access.
