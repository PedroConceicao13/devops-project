# DevOps Project - Automated Site Deployment to EC2

## Description
This project implements a CI/CD pipeline using GitHub Actions to automate the build, push of a Docker image to Amazon ECR, and deployment of a static website to an EC2 instance on AWS. The site consists of HTML, CSS, and JavaScript files, containerized with Docker. The pipeline is triggered on pushes to the `main` branch, ensuring continuous integration and delivery with a focus on security and efficiency, following DevSecOps best practices such as temporary credentials via OIDC and managed secrets.

This approach promotes scalability through implied Infrastructure as Code (IaC) in the pipeline configuration, and reliability through job dependencies to prevent failed deployments. For more details on GitHub Actions, see the [official documentation](https://docs.github.com/en/actions).

## Video Tutorial
Watch the complete video tutorial explaining the CI/CD pipeline implementation step by step, from initial configuration to EC2 deployment, focusing on DevOps best practices like temporary credentials via OIDC and dynamic Docker image tagging.

This video is part of the "DevOps in Practice" series, corresponding to part 4 of a comprehensive playlist covering the entire process from initial concepts to advanced optimizations, including integration with tools like Terraform and SonarQube. Access the full playlist here: [DevOps in Practice Playlist](https://youtube.com/playlist?list=PLOCRt8ucq6xNSMUvfTxnk-M4mk9Etwpy6&si=LOoW5N9Xc6-QViKN).

## Prerequisites

- AWS account with permissions for ECR and EC2
- GitHub repository with configured secrets: `INSTANCE_KEY` (private SSH key) and `PUBLIC_IP` (EC2 instance public IP)
- IAM Role in AWS for GitHub Actions (using OIDC for secure authentication without permanent access keys). See [AWS documentation for GitHub Actions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html)
- EC2 instance with Docker installed and SSH access configured for `ec2-user`
- ECR repository created in `us-east-2` region

## Repository Structure

- **website/**: Contains site files:
  - `index.html`: Main page
  - CSS and JavaScript files for styling and functionality
- **Dockerfile**: File for building Docker image of the site, based on a web server image like Nginx (example: copies files to `/usr/share/nginx/html`)
- **.github/workflows/deploy.yaml**: GitHub Actions workflow for CI/CD pipeline

## CI/CD Pipeline

The pipeline is defined in the `deploy.yaml` file and consists of two sequential jobs: build and push to ECR, followed by deployment via SSH on EC2.

```yaml
name: Pipeline CI/CD

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  job1:
    name: build_ecr
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5.0.0

      - name: "Configure AWS Credentials" 
        uses: aws-actions/configure-aws-credentials@v4.3.1
        with:
          role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/GitHubActionsRepoApp
          aws-region: us-east-2

      - name: Amazon ECR "Login" Action for GitHub Actions
        uses: aws-actions/amazon-ecr-login@v2.0.1
      
      - name: Build, Tag, and Push image to Amazon ECR
        run: |
          docker build -t meu-website:v1.0 .
          docker tag meu-website:v1.0 <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/site_prod:v1.0
          docker push <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/site_prod:v1.0

  job2:
    name: deploy_ec2
    needs: job1
    env:
      INSTANCE_KEY: ${{secrets.INSTANCE_KEY}}
      PUBLIC_IP: ${{secrets.PUBLIC_IP}}
    runs-on: ubuntu-latest
    steps:
     - name: Deploy EC2 SSH
       run: |
          echo "$INSTANCE_KEY" > chave-site.pem
          chmod 400 chave-site.pem
          ssh -i chave-site.pem -o StrictHostKeyChecking=no ec2-user@$PUBLIC_IP << EOF
            aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com
            docker pull <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/site_prod:v1.0
            echo "parando container antigo site"
            docker stop site || true
            echo "removendo container antigo site"
            docker rm site || true
            echo "rodando nova tag da imagem"
            docker run -d -p 80:80 --name site <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/site_prod:v1.0
            echo "Parabéns! Imagem foi atualizada!"
            docker ps
          EOF
          rm chave-site.pem
```

**Security Notes:**
- Replace `<ACCOUNT_ID>` with your AWS account ID
- Use GitHub secrets to store sensitive keys, avoiding exposure in code
- The `-o StrictHostKeyChecking=no` option is used for automation

## Step Explanation

1. **Trigger**: Pipeline is automatically triggered on pushes to `main` branch

2. **Job 1 - build_ecr**:
   - **Checkout**: Downloads repository code
   - **Configure AWS Credentials**: Assumes IAM role via OIDC for secure authentication without access keys
   - **ECR Login**: Logs into ECR using temporary credentials
   - **Build, Tag and Push**: Builds Docker image from `Dockerfile`, tags with version and sends to ECR repository

3. **Job 2 - deploy_ec2** (depends on Job 1):
   - Uses secrets for SSH key and EC2 IP
   - Creates temporary file with SSH key and configures permissions
   - Connects via SSH to EC2 instance and runs commands to:
     - Login to ECR
     - Pull new image
     - Stop and remove old container (if exists)
     - Run new container mapping port 80
     - Verify with `docker ps`
   - Removes temporary key to prevent leaks



## Customizable CI/CD Pipeline (deploy2.yaml)

### Description

This is a customizable and slightly more complex version of the CI/CD pipeline, defined in the `deploy2.yaml` file. It improves automation of Docker image build, tag, and push to Amazon ECR, followed by deployment to an EC2 instance via SSH. Improvements include: dynamic image tagging based on commit SHA and environment (prod or dev, depending on branch), use of outputs to pass variables between jobs, and greater flexibility for multi-branch environments. This follows DevSecOps best practices like temporary credentials via OIDC, environment variables for dynamic configuration, and atomicity via job dependencies, reducing toil and improving traceability (e.g., logs with unique tags). For details on outputs in GitHub Actions, see the [official documentation](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idoutputs).

This version is ideal for scenarios requiring scalability, such as integration with multiple environments or integration with tools like Terraform for dynamic resource provisioning. It maintains focus on security, using managed secrets and minimal permissions (`contents: read` and `id-token: write`).

### Prerequisites

Same as original pipeline, with addition of:
- Configuration of additional branches (e.g., `dev` for test environments), adjusting `ENVIRONMENT` logic
- ECR repository with permissions for pushing dynamic tags

### Customizable CI/CD Pipeline

The workflow is triggered on pushes to `main`, but can be expanded to other branches. Here's the content of `deploy2.yaml` (with sensitive data replaced by placeholders):

```yaml
name: Pipeline CI/CD

on:
  push:
    branches:
      - main
permissions:
  contents: read
  id-token: write

jobs:
  build-ecr:
    runs-on: ubuntu-latest
    outputs:
      registry: ${{ steps.login-ecr.outputs.registry }}
      image_tag: ${{ steps.build.outputs.image_tag }}
      image_uri: ${{ steps.build.outputs.image_uri }}
    steps:
      - uses: actions/checkout@v5.0.0

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4.3.1
        with:
          role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/gitHubActionsECR
          aws-region: us-east-2

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push docker image to Amazon ECR
        id: build
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: site_prod
          ENVIRONMENT: ${{ github.ref_name == 'main' && 'prod' || 'dev' }}
        run: |
          COMMIT_SHA=$(echo $GITHUB_SHA | cut -c1-7)
          IMAGE_TAG=${ENVIRONMENT}-${COMMIT_SHA}
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
          echo "image_tag=$IMAGE_TAG" >> $GITHUB_OUTPUT
          echo "image_uri=$REGISTRY/$REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy-ssh:
    runs-on: ubuntu-latest
    needs: build-ecr
    steps:
      - name: Deploy to EC2 via SSH
        env:
          IMAGE_TAG: ${{ needs.build-ecr.outputs.image_tag }}
          IMAGE_URI: ${{ needs.build-ecr.outputs.image_uri }}
          REGISTRY: ${{ needs.build-ecr.outputs.registry }}
          INSTANCE_KEY: ${{ secrets.INSTANCE_KEY }}
          ELASTIC_IP: ${{ secrets.ELASTIC_IP }}
        run: |
          echo $IMAGE_URI
          echo "$INSTANCE_KEY" > key.pem
          chmod 400 key.pem
          ssh -i key.pem -o StrictHostKeyChecking=no ec2-user@$ELASTIC_IP << EOF
            aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin $REGISTRY
            echo "baixando imagem $IMAGE_TAG"
            docker pull $IMAGE_URI
            echo "parando container antigo e iniciando novo"
            docker stop site || true
            docker rm site || true
            echo "iniciando container novo com a imagem $IMAGE_TAG"
            docker run -d -p 80:80 --name site $IMAGE_URI
            docker ps
            echo "deploy finalizado"
          EOF
          rm key.pem
```

### Step Explanation

1. **Trigger and Permissions**: Triggered on pushes to `main`, with minimal permissions for content reading and OIDC ID token generation

2. **Job build-ecr**:
   - **Checkout**: Downloads code
   - **Configure AWS Credentials**: Assumes IAM role via OIDC
   - **Login to ECR**: Generates temporary credentials and exposes `registry` as output
   - **Build, Tag and Push**: Defines environment dynamically (`prod` for `main`, `dev` otherwise), generates unique tag with commit SHA for immutable versioning, builds and pushes image. Outputs (`image_tag`, `image_uri`) are passed to next job, promoting decoupling

3. **Job deploy-ssh** (depends on build-ecr):
   - Uses outputs from previous job for dynamic image
   - Creates temporary SSH key with secure permissions
   - Via SSH on EC2: Logs into ECR, pulls specific image, stops/removes old container, starts new one mapping port 80, verifies with `docker ps`
   - Removes key to prevent exposure

This structure improves reliability with immutable tags (facilitating rollbacks) and scalability for multi-environments, aligning with SRE principles.

### Improvements Over Original Version

- **Dynamic Tagging**: Uses commit SHA for unique tags, avoiding overwrites and enabling traceability (e.g., rollback to specific tag via `docker pull`)
- **Customizable Environments**: Conditional logic for `prod/dev`, expandable to more branches with GitOps tools
- **Outputs Between Jobs**: Improves modularity, facilitating addition of jobs (e.g., tests with SonarQube before deploy)
- **Flexibility**: Easy integration with IaC (e.g., Terraform to create ECR/EC2) or monitoring tools

For local testing, build with `docker build -t <registry>/<repo>:prod-<sha> .` and simulate manual deployment.