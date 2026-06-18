# Host an HTML Website on AWS EC2

A foundational AWS project demonstrating three progressively automated methods 
of deploying a static HTML website on an EC2 instance — starting with manual 
SSH commands and ending with fully automated User Data scripting.

---

## 📋 Overview

This project served as the foundation for understanding how cloud servers are 
provisioned, configured, and made publicly accessible on AWS. Rather than jumping 
straight into complex infrastructure, each task built deliberately on the previous 
one — showing a clear progression from manual to fully automated deployments.

**Key concepts covered:**
- Launching and connecting to EC2 instances
- Configuring Security Groups as firewall rules
- Installing and running Apache as a web server
- Automating server setup with User Data scripts
- Hosting files from S3 and GitHub

---

## 🛠️ Tools & Technologies

| Category | Tools |
|---|---|
| **Compute** | Amazon EC2 |
| **Storage** | Amazon S3 |
| **Web Server** | Apache HTTP Server (`httpd`) |
| **Scripting** | Bash |
| **Access** | SSH, Key Pairs |
| **Security** | AWS Security Groups |
| **Source Control** | GitHub |

---

## 🚀 Three Deployment Methods

### Task 1 — Manual SSH Deployment

The most hands-on approach: launch an EC2 instance, SSH in, and run commands 
manually to install Apache and pull the website files from GitHub.

**Steps:**
1. Select region: **US East (N. Virginia)**
2. Create a **Security Group** — open port 80 (HTTP, anywhere) and port 22 
   (SSH, My IP only)
3. Launch EC2 instance with the Security Group and a key pair
4. SSH into the instance:
```bash
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@YOUR-PUBLIC-IP
```
5. Run the setup script inside the instance:
```bash
# Switch to root user
sudo -i
# Update all packages
yum update -y
# Install Apache web server
yum install -y httpd
# Navigate to web root
cd /var/www/html
# Download website files from GitHub
wget https://github.com/azeezsalu/techmax/archive/refs/heads/main.zip
# Unzip the files
unzip main.zip
# Copy files to the web root
cp -r techmax-main/* /var/www/html/
# Clean up
rm -rf techmax-main main.zip
# Enable and start Apache
systemctl enable httpd
systemctl start httpd
```
6. Visit the **public IPv4 address** in a browser to verify the site is live

---

### Task 2 — S3-Assisted Deployment (User Data Script)

Upload the website zip to a public S3 bucket, then automate the entire 
deployment using an EC2 **User Data** script — the server configures itself 
on first boot, no SSH required.

**Steps:**
1. Download the web files and upload the zip to a public S3 bucket
2. Add a **bucket policy** allowing public read access
3. Launch a new EC2 instance and paste the following into the **User Data** 
   field under Advanced Details:
```bash
#!/bin/bash
sudo su
yum update -y
yum install -y httpd
cd /var/www/html
wget https://bj-htmlwebsite-project.s3.amazonaws.com/mole.zip
unzip mole.zip
cp -r mole-main/* /var/www/html/
rm -rf mole-main main.zip
systemctl enable httpd
systemctl start httpd
```
4. Visit the public IPv4 address to confirm the site is live — no SSH needed

---

### Task 3 — GitHub-Based Deployment (User Data Script)

Same automation approach as Task 2, but files are pulled directly from a 
**public GitHub repository** instead of S3 — no manual bucket setup required.

> ⚠️ **Important:** Always use the **raw** GitHub URL in the `wget` command, 
> not the standard GitHub page URL. The standard URL returns an HTML page, 
> not the actual file — the download will silently fail.

```bash
#!/bin/bash
sudo -i
yum update -y
yum install -y httpd
cd /var/www/html
wget https://raw.githubusercontent.com/bconway1906/htmlfiles/main/mole.zip
unzip mole.zip
cp -r mole-main/* /var/www/html/
rm -rf mole-main main.zip
systemctl start httpd
systemctl enable httpd
```

---

## 💡 Key Lessons Learned

- **Security Groups are your first line of defense.** Limiting SSH to My IP only 
  (instead of anywhere) is a critical security habit — if port 22 is open to the 
  world, bots will attempt to brute-force your server within minutes.
- **User Data scripts eliminate manual setup entirely.** The server installs 
  Apache, downloads the site, and starts serving traffic automatically on first 
  boot — this is the foundation of infrastructure automation and scales directly 
  to tools like Terraform and Ansible.
- **Raw GitHub URLs are not the same as the page URL.** `wget` needs the actual 
  file content, not the HTML wrapper GitHub shows in the browser. Always use 
  `raw.githubusercontent.com` format.
- **Each task built on the previous one intentionally.** Manual → S3 → GitHub 
  shows the natural progression from understanding what's happening to automating 
  it — the same progression you see in real-world DevOps work.

---

## 🧹 Cost Management

This project uses a single **EC2 t2.micro** instance — free tier eligible. 
The only resources that could incur charges beyond the free tier are S3 storage 
(Task 2) and data transfer. After each task, terminate the instance to avoid 
ongoing compute charges.
