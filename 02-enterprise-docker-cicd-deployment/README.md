# Enterprise Deployment: Docker + ECR + EC2 + GitHub Actions

### Prerequisite

- Start your EC2 instance on AWS
  - Security Groups must include: HTTP/SSH
  - Machine: ubuntu
- Create an AWS ECR repository
- Install Docker on EC2
- Push your code to GitHub

**Final CI/CD Flow**

```
GitHub (production-grade-backend-repo)
	↓
GitHub Actions
	↓
Docker build
	↓
Push image → AWS ECR
	↓
SSH → EC2
	↓
Docker pull + run (prod)
```

<br><br><br>

## Step 0: Connect to EC2 directly with aws CMD

- EC2 → Instances → `Connect`
- Open `EC2 Instance Connect`
- Hit `connect` button at the bottom

It will open another tab with aws terminal.

<br><br><br>

### Step 1: Install Docker & AWS CLI in EC2 Terminal

Update and install Docker and aws CLI:

```bash
sudo apt update   # system update
```

```bash
sudo apt install docker.io -y
```

```bash
docker --version   # check if installed or not
```

```bash
sudo usermod -aG docker ubuntu  # docker group fix
```

```bash
exit     # upper wale cmd k baad logout mandatory
```

```bash
sudo apt install unzip curl -y
```

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```bash
unzip awscliv2.zip
```

```bash
sudo ./aws/install
```

```bash
aws --version   # check if installed or not
```

<br><br><br>

## Step 2: Create AWS ECR Repository

In AWS Console → ECR → Create repository:

- Name: your-backend-repo
- Visibility: Private

Note: Here, your<br> ECR_REGISTRY:
`<aws_account_id>.dkr.ecr.<region>.amazonaws.com`
<br>
ECR_REPOSITORY:`your ecr-repo name`

<br><br><br>

## Step 3: Attach IAM role to EC2

1. create IAM role first:

   - AWS Console → IAM → Roles → Create Role
   - Select AWS Service → then in dropdown choose `EC2`
   - Attach policy: `AmazonEC2ContainerRegistryReadOnly`
   - Role name: `EC2-ECR-Access-Role` (kuch v add kr skte ho)
   - Create Role

2. EC2 Instance ko Role attach karo:
   - EC2 Console → Instances
   - Apna instance select karo → `Actions` → `Security` → `Modify IAM Role`
   - Dropdown se apna role select karo `(EC2-ECR-Access-Role)`
   - Update IAM Role click karo

#### To test this step 3 is working or not, do this:

- Go to EC2 terminal directly via connect.
- Paste this there in the terminal

```bash
aws ecr get-login-password --region ap-south-1
```

- if password returns, then its working ✅

<br><br><br>

## Step 4: Add GitHub Secrets

`Repo → Settings → Secrets and variables → Actions`

Add these secrets:
| Name | Value |
|--------------------|--------------------------------------------|
| AWS_ACCESS_KEY_ID | Your IAM access key |
| AWS_SECRET_ACCESS_KEY | Your IAM secret |
| AWS_REGION | e.g. us-east-1 |
| ECR_REPOSITORY | your-ecr-repo-name |
| ECR_REGISTRY | <aws_account_id>.dkr.ecr.<region>.amazonaws.com |
| EC2_HOST | Your EC2 public IP |
| EC2_USER | ubuntu |
| EC2_SSH_KEY | Paste contents of your .pem file |

<br>

If your project also contains some `.env` you can also add it here:
| Name | Value |
|--------------------|--------------------------------------------|
| PORT | your server port number |
| MONGODB_URI | your db url |

<br><br><br>

## Step 5: Create workflow file:

Create workflow file: `.github/workflows/deploy.yml`

```yaml
name: Production CI/CD - Backend  # any name you can give

on:
  push:
    branches:
      - main  # Jab bhi 'main' branch pe push hoga, ye workflow automatically trigger hoga

jobs:                          # Jobs Definition
  build-deploy:  			   # Job ka naam
    runs-on: ubuntu-latest
    environment: production

    steps:
	# STEP 1: Source code checkout
      - name: Checkout source
        uses: actions/checkout@v4 # GitHub repo ka code download karta hai runner pe
		# Ye step zaroori hai taaki baaki steps code access kar sake



	# STEP 2: AWS authentication setup
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }} # IAM User ka Access Key
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }} # IAM User ka Secret Key
          aws-region: ${{ secrets.AWS_REGION }} # AWS region (e.g., ap-south-1)



	# STEP 3: ECR (Elastic Container Registry) login
      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2 # Docker ko ECR ke saath authenticate karta hai
		# Iske baad docker push/pull ECR se kar sakte ho



	# STEP 4: Docker image build aur ECR pe push
      - name: Build & Push Docker image
        uses: docker/build-push-action@v5  # Docker image build aur push karne ka action
        with:
          context: .
          push: true # Image build hone ke baad ECR pe push karo
          tags: |    # Image ko 2 tags ke saath push karo:
            ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest # 1. 'latest' tag (hamesha latest version)
            ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:${{ github.sha }} # 2. Git commit SHA tag (specific version tracking)



	 # STEP 5: EC2 pe deployment via SSH
      - name: Deploy to EC2 (Docker)
        uses: appleboy/ssh-action@v1.0.0  # EC2 instance mein SSH karke commands run karta hai
        with:
          host: ${{ secrets.EC2_HOST }} # EC2 ka public IP address (e.g., 52.66.184.241)
          username: ${{ secrets.EC2_USER }} # SSH username (usually 'ubuntu')
          key: ${{ secrets.EC2_SSH_KEY }} # Private SSH key (.pem file ka content)
          script: |
            set -e # Agar koi command fail ho, immediately stop karo (error handling)

			# ECR se login karo (EC2 pe Docker ko authenticate karna hai)
            aws ecr get-login-password --region ${{ secrets.AWS_REGION }} \
              | docker login --username AWS --password-stdin ${{ secrets.ECR_REGISTRY }}

			# Latest Docker image pull karo ECR se
            docker pull ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest


            docker stop backend-prod || true # Purana container stop karo (agar chal raha ho)
            docker rm backend-prod || true # Purana container delete karo



			# Environment variables file banao temporary location pe
            cat <<EOF > /tmp/.env   # 'cat <<EOF' se multi-line file create kar rahe ho
            NODE_ENV=production
            PORT=${{ secrets.PORT }}
            MONGODB_URI=${{ secrets.MONGODB_URI }}
            EOF
			# Ye file /tmp/.env location pe create hogi with all secrets


			# Naya container run karo
            docker run -d \  # '-d' = detached mode (background mein chalega)
              --name backend-prod \  # Container ka naam 'backend-prod'
              -p 80:8080 \  # Port mapping: External 80 -> Internal 8080 (browser se 80 pe access, app 8080 pe listen)
              --env-file /tmp/.env \  # Environment variables /tmp/.env file se load karo
              --restart always \  # Container crash ho ya EC2 restart ho, automatically restart karo
              ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest # Image jo abhi pull ki hai use karo

            # Security: Environment file delete karo (sensitive data hai)
            rm /tmp/.env  # Temporary env file ko remove karo
```

#### Clean Version of above file

```bash
name: Production CI/CD - Backend

on:
  push:
    branches:
      - main

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build & Push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest
            ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:${{ github.sha }}

      - name: Deploy to EC2 (Docker)
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            set -e

            aws ecr get-login-password --region ${{ secrets.AWS_REGION }} \
              | docker login --username AWS --password-stdin ${{ secrets.ECR_REGISTRY }}

            docker pull ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest

            docker stop backend-prod || true
            docker rm backend-prod || true

            cat <<EOF > /tmp/.env
            NODE_ENV=production
            PORT=${{ secrets.PORT }}
            MONGODB_URI=${{ secrets.MONGODB_URI }}
            EOF

            docker run -d \
              --name backend-prod \
              -p 80:8080 \
              --env-file /tmp/.env \
              --restart always \
              ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest

            rm /tmp/.env
```

<br><br><br>

## Step 6: Create a Docker file in your repo `(Dockerfile)`

```bash
# --------------------
# Build stage
# --------------------
FROM node:22-alpine AS builder

WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install all deps (including dev for tsc)
RUN npm install

# Copy source code
COPY . .

# Build TypeScript -> dist/
RUN npm run build


# --------------------
# Production stage
# --------------------
FROM node:22-alpine

WORKDIR /app

ENV NODE_ENV=production

# Copy only package files
COPY package*.json ./

# Install ONLY production deps
RUN npm install --omit=dev

# Copy compiled JS from builder
COPY --from=builder /app/dist ./dist

EXPOSE 8080

CMD ["node", "dist/server.js"]
```

## <br><br><br>

CICD pipeline setup - done 🎉🎉🎉🎉🎉🎉🎉

---

<br>

## FINAL STEP : Push to `main` now

Push changes to `main`. Workflow will:

- Build Docker image
- Push to ECR
- SSH to EC2
- Pull latest image and run container

🎉 Ho gaya! Ab koi bhi change push karo main branch me, automatic Docker deploy ho jayega!

---

## To verify your deployment is live or not:

Go to EC2 instance, open the public ip in the browswer with correct endpoint.

Example: http://52.66.184.241/api/v1
<br>
<br>
<br>
<br>
<br>

<br>

# ⚠️ Quick Debug Commands

#### EC2 Terminal

- Check running containers:

```bash
docker ps
```

- Check logs:

```bash
docker logs -f backend
```

- Restart container:

```bash
docker restart backend
```

- Remove container:

```bash
docker stop backend && docker rm backend
```

- Remove old images:

```bash
docker image prune -f
```

---
