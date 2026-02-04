# 🚀 DevOps Lab - Project 1: Docker Containerization & Manual AWS Deploy

## 📋 Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Project Architecture](#project-architecture)
4. [Phase 1: Local Environment Setup](#phase-1-local-environment-setup)
5. [Phase 2: Containerization with Docker](#phase-2-containerization-with-docker)
6. [Phase 3: Local Container Testing](#phase-3-local-container-testing)
7. [Phase 4: Amazon ECR Configuration](#phase-4-amazon-ecr-configuration)
8. [Phase 5: Push Image to ECR](#phase-5-push-image-to-ecr)
9. [Phase 6: EC2 Instance Provisioning](#phase-6-ec2-instance-provisioning)
10. [Phase 7: Deploy to EC2](#phase-7-deploy-to-ec2)
11. [Verification and Testing](#verification-and-testing)
12. [Troubleshooting](#troubleshooting)
13. [Resource Cleanup](#resource-cleanup)

---

## 🎯 Overview

### What You'll Build
Containerize a static website (HTML, CSS, JavaScript) using Docker and deploy it manually to an EC2 instance on AWS using Amazon ECR for image management.

### Why This Matters
- **Portability**: Your site works the same in any environment
- **Isolation**: Eliminates "works on my machine" problems
- **Scalability**: Foundation for more complex implementations
- **Industry Standard**: Docker is widely used in production

### Estimated Time: 2-3 hours

---

## 🔧 Prerequisites

### Required Tools

#### 1. **Docker Desktop**
- **Windows/Mac**: Download from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- **Linux**: Install via terminal:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Verify installation:
```bash
docker --version
```

#### 2. **AWS CLI**
Install following [official documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Verify:
```bash
aws --version
```

#### 3. **AWS Account**
- Create free account at [aws.amazon.com](https://aws.amazon.com)
- ⚠️ **Important**: Some resources may incur costs. Use Free Tier when possible

#### 4. **Code Editor**
- Recommended: [Visual Studio Code](https://code.visualstudio.com/)
- Useful extensions: Docker, AWS Toolkit

### Project Structure
```
my-project/
├── website/
│   ├── index.html
│   ├── styles.css
│   ├── script.js
│   └── assets/
│       └── (images, fonts, etc.)
└── Dockerfile (we'll create this)
```

---

## 🏗️ Project Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Local Code     │────▶│   Docker Image  │────▶│   Amazon ECR    │
│  (HTML/CSS/JS)  │     │   (Container)   │     │   (Registry)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │    Browser      │◀────│   Amazon EC2    │
                        │  (User Access)  │     │   (Container)   │
                        └─────────────────┘     └─────────────────┘
```

---

## 📦 Phase 1: Local Environment Setup

### Step 1.1: Verify Project Structure

Navigate to your project directory:
```bash
cd path/to/your/project
ls -la
```

Check the `website/` folder:
```bash
ls -la website/
```

### Step 1.2: Test Website Locally (Optional)

Open `index.html` directly in your browser:
```bash
# Mac
open website/index.html

# Linux
xdg-open website/index.html

# Windows (PowerShell)
start website/index.html
```

---

## 🐳 Phase 2: Containerization with Docker

### Step 2.1: Create the Dockerfile

In the project root (same level as `website/` folder), create a file named `Dockerfile`:

```bash
touch Dockerfile
```

### Step 2.2: Write the Dockerfile

Open Dockerfile in your editor and add:

```dockerfile
# Base image - Nginx Alpine (lightweight and efficient)
FROM nginx:alpine

# Copy website files to Nginx directory
COPY website/ /usr/share/nginx/html/

# Expose port 80 (documentation - doesn't actually open the port)
EXPOSE 80

# Default command when container starts
CMD ["nginx", "-g", "daemon off;"]
```

#### 🎓 Understanding Each Line:

- **FROM nginx:alpine**: Defines base image. Alpine is a super lightweight Linux version
- **COPY**: Copies files from host into the image
- **EXPOSE**: Documents which port the container uses
- **CMD**: Defines default command when starting the container

### Step 2.3: Build the Docker Image

In the terminal, from project root, run:

```bash
docker build -t my-website:v1.0 .
```

#### 🎓 Understanding the Command:
- **docker build**: Command to build an image
- **-t my-website:v1.0**: Tag (name:version) of the image
- **.**: Build context (current directory)

Expected output:
```
[+] Building 10.5s (8/8) FINISHED
 => [internal] load build definition from Dockerfile
 => transferring dockerfile: 370B
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:alpine
 => [1/3] FROM docker.io/library/nginx:alpine
 => [2/3] RUN rm -rf /usr/share/nginx/html/*
 => [3/3] COPY website/ /usr/share/nginx/html/
 => exporting to image
 => naming to docker.io/library/my-website:v1.0
```

### Step 2.4: Verify Created Image

```bash
docker images
```

You should see:
```
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
my-website     v1.0      abc123def456   30 seconds ago   23.5MB
```

---

## 🧪 Phase 3: Local Container Testing

### Step 3.1: Run Container Locally

```bash
docker run -d -p 8080:80 --name my-website-container my-website:v1.0
```

#### 🎓 Understanding the Command:
- **docker run**: Creates and runs a container
- **-d**: Runs in background (detached)
- **-p 8080:80**: Maps host port 8080 to container port 80
- **--name**: Container name
- **my-website:v1.0**: Image to use

### Step 3.2: Verify Container is Running

```bash
docker ps
```

Expected output:
```
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                  NAMES
xyz789abc123   my-website:v1.0  "nginx -g 'daemon..."   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   my-website-container
```

### Step 3.3: Test in Browser

Open your browser and go to:
```
http://localhost:8080
```

Your website should be working! 🎉

### Step 3.4: View Container Logs (Optional)

```bash
docker logs my-website-container
```

### Step 3.5: Stop and Remove Test Container

```bash
# Stop container
docker stop my-website-container

# Remove container
docker rm my-website-container
```

---

## ☁️ Phase 4: Amazon ECR Configuration

### Step 4.1: Access AWS Console

1. Go to [console.aws.amazon.com](https://console.aws.amazon.com)
2. Log in with your credentials

### Step 4.2: Navigate to ECR

1. In the top search bar, type "ECR"
2. Click on "Elastic Container Registry"

### Step 4.3: Create Repository

1. Click "Create repository"
2. Configure:
   - **Visibility settings**: Private
   - **Repository name**: `my-website`
   - **Tag immutability**: Disabled (default)
   - **Scan on push**: Enabled (recommended for security)
3. Click "Create repository"

### Step 4.4: Note the Repository URI

After creation, you'll see something like:
```
123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website
```

⚠️ **Important**: Copy and save this URI – you'll need it!

---

## 📤 Phase 5: Push Image to ECR

### Step 5.1: Configure AWS CLI

If not already configured, run:
```bash
aws configure
```

Provide:
- **AWS Access Key ID**: From IAM
- **AWS Secret Access Key**: From IAM
- **Default region**: e.g., us-east-1
- **Default output format**: json

### Step 5.2: Authenticate Docker with ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

⚠️ **Replace**: 
- `us-east-1` with your region
- `123456789012` with your Account ID

Expected output:
```
Login Succeeded
```

### Step 5.3: Tag Image for ECR

```bash
docker tag my-website:v1.0 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website:v1.0
```

### Step 5.4: Push Image

```bash
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website:v1.0
```

Expected output:
```
The push refers to repository [123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website]
abc123: Pushed
def456: Pushed
v1.0: digest: sha256:xyz789... size: 1234
```

### Step 5.5: Verify in AWS Console

1. Return to ECR in AWS console
2. Click on your repository
3. You should see the image with tag v1.0

---

## 🖥️ Phase 6: EC2 Instance Provisioning

### Step 6.1: Navigate to EC2

1. In AWS console, search for "EC2"
2. Click on "EC2"

### Step 6.2: Launch Instance

1. Click "Launch Instance"
2. Configure:

#### Name and Tags
- **Name**: `my-website-server`

#### Application and OS Image
- **AMI**: Amazon Linux 2023 (Free tier eligible)

#### Instance Type
- **Instance type**: t2.micro (Free tier eligible)

#### Key Pair
- Click "Create new key pair"
- **Key pair name**: `my-website-key`
- **Key pair type**: RSA
- **Private key file format**: .pem (Linux/Mac) or .ppk (Windows/PuTTY)
- Click "Create key pair" and save the file

⚠️ **IMPORTANT**: Keep this file secure! You'll need it to access EC2.

#### Network Settings
- **VPC**: Default
- **Subnet**: No preference
- **Auto-assign public IP**: Enable
- **Firewall (security groups)**: Create security group
  - **Security group name**: `my-website-sg`
  - **Description**: Security group for website

#### Security Group Rules
Add these rules:

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| SSH  | TCP      | 22         | My IP  |
| HTTP | TCP      | 80         | 0.0.0.0/0 |

#### Configure Storage
- **Volume**: 8 GiB gp3 (default)

### Step 6.3: Configure IAM Role (ECR Permissions)

#### Create IAM Role
1. In "Advanced details", find "IAM instance profile"
2. Click "Create new IAM profile"
3. Or go to IAM Console:
   - Click "Roles" → "Create role"
   - **Trusted entity**: AWS service
   - **Use case**: EC2
   - **Permissions**: Add `AmazonEC2ContainerRegistryReadOnly`
   - **Role name**: `EC2-ECR-Role`
4. Return to EC2 configuration and select the created role

### Step 6.4: Review and Launch

1. Review all configurations
2. Click "Launch instance"
3. Wait for instance to initialize (status: running)

### Step 6.5: Note Important Information

Record:
- **Public IP**: e.g., 54.123.45.67
- **Instance ID**: e.g., i-0abc123def456789

---

## 🚀 Phase 7: Deploy to EC2

### Step 7.1: Connect to EC2 Instance

#### On Linux/Mac:
```bash
# Adjust key permissions
chmod 400 my-website-key.pem

# Connect via SSH
ssh -i my-website-key.pem ec2-user@54.123.45.67
```

#### On Windows (using PuTTY):
1. Convert .pem key to .ppk using PuTTYgen
2. Use PuTTY to connect with .ppk key

Expected output:
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-172-31-xx-xx ~]$
```

### Step 7.2: Install Docker on EC2

```bash
# Update packages
sudo yum update -y

# Install Docker
sudo yum install docker -y

# Start Docker service
sudo systemctl start docker

# Enable Docker on boot
sudo systemctl enable docker

# Add ec2-user to docker group
sudo usermod -a -G docker ec2-user

# Verify installation
docker --version
```

### Step 7.3: Logout and Login Again

```bash
# Exit
exit

# Reconnect
ssh -i my-website-key.pem ec2-user@54.123.45.67
```

### Step 7.4: Authenticate Docker with ECR on EC2

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

### Step 7.5: Pull Image from ECR

```bash
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website:v1.0
```

Expected output:
```
v1.0: Pulling from my-website
Status: Downloaded newer image for 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website:v1.0
```

### Step 7.6: Run the Container

```bash
docker run -d -p 80:80 --name my-website-prod --restart always 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website:v1.0
```

#### 🎓 Important Parameters:
- **--restart always**: Restarts container if EC2 restarts
- **-p 80:80**: Maps port 80 (default HTTP)

### Step 7.7: Verify Running

```bash
# Check container
docker ps

# Check logs
docker logs my-website-prod
```

---

## ✅ Verification and Testing

### Test 1: Access via Browser

1. Open your browser
2. Enter the EC2 public IP: `http://54.123.45.67`
3. Your website should appear! 🎉

### Test 2: Check Logs on EC2

```bash
# Container logs
docker logs -f my-website-prod

# Container stats
docker stats my-website-prod
```

### Test 3: Test Restart

```bash
# Stop container
docker stop my-website-prod

# Verify it stopped
docker ps

# Start again
docker start my-website-prod

# Verify it's back
docker ps
```

---

## 🔧 Troubleshooting

### Problem 1: "Cannot connect to the Docker daemon"

**Solution**:
```bash
sudo systemctl start docker
sudo usermod -a -G docker $USER
# Logout and login again
```

### Problem 2: Website Doesn't Open in Browser

**Checks**:
1. Is port 80 open in Security Group?
2. Is container running? (`docker ps`)
3. Is public IP correct?
4. Test with curl on EC2: `curl localhost`

### Problem 3: "No basic auth credentials" when Pulling from ECR

**Solution**:
```bash
# Re-authenticate
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [ECR_URI]
```

### Problem 4: Permission Denied in Docker

**Solution**:
```bash
# Add user to docker group
sudo usermod -a -G docker ec2-user
# Logout and login
exit
ssh -i key.pem ec2-user@IP
```

---

## 🧹 Resource Cleanup

⚠️ **IMPORTANT**: Clean up resources after the lab to avoid charges!

### Step 1: Stop and Remove Container on EC2

```bash
docker stop my-website-prod
docker rm my-website-prod
docker rmi 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-website:v1.0
```

### Step 2: Terminate EC2 Instance

1. AWS Console → EC2
2. Select your instance
3. Actions → Instance State → Terminate

### Step 3: Delete ECR Image

1. AWS Console → ECR
2. Select repository
3. Select image
4. Delete

### Step 4: Delete ECR Repository (Optional)

1. Select repository
2. Delete

### Step 5: Delete Security Group

1. EC2 → Security Groups
2. Select `my-website-sg`
3. Actions → Delete

### Step 6: Delete IAM Role (Optional)

1. IAM → Roles
2. Select `EC2-ECR-Role`
3. Delete

---

## 🎓 Concepts Learned

✅ **Containerization**: Packaging applications with dependencies

✅ **Docker**: Platform for creating and running containers

✅ **Dockerfile**: Configuration file for building images

✅ **ECR**: Private Docker image registry on AWS

✅ **EC2**: Virtual machines in AWS cloud

✅ **Security Groups**: Virtual firewall for EC2

✅ **IAM Roles**: Permission management in AWS

---

## 🚀 Next Steps

After completing this lab, you're ready for:

1. **Project 2**: Automate this process with CI/CD
2. **Project 3**: Use Terraform for Infrastructure as Code
3. **Explore**: Docker Compose, Kubernetes, ECS/Fargate

---

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [AWS ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Best Practices for Dockerfile](https://docs.docker.com/develop/dev-best-practices/)

---

**Congratulations! 🎉** You completed the containerization and manual AWS deployment lab!

Developed with ❤️ for the DevOps journey