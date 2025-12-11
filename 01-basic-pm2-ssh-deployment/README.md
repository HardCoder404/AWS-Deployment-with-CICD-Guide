# Basic Deployment PM2 and SSH Guide

### Prerequisite
- Start your EC2 instance on AWS
  * Security Groups must include: HTTP/SSH
  * Machine: ubuntu
- Push your code to github



## Step 0: 
* Go to inside your EC2 > instances.
* Click on the `connect` button at the top right side.
* It will open new tab with `Connect to instance` tag:
  -  go to the `SSH client` tab.
  -  copy the example url: it looks like this: 
  - `ssh -i "ts-production-setup-backend-key-pair.pem" ubuntu@ec2-98-92-123-244.compute-1.amazonaws.com`

<br><br>

## Step 1: 
#### PowerShell Terminal (Your Windows Machine)
* Check weather in which folder you have installed your `.pem` file. 
* [ mostly, hum download folders k andr hi karte hai ]
* Open the terminal and go to that folder via `cd` and paste your copied `SSH client`.

```bash
cd Downloads
ssh -i "ts-production-setup-backend-key-pair.pem" ubuntu@ec2-98-92-123-244.compute-1.amazonaws.com
```

<br><br>


## Step 2:
#### EC2 Terminal (After SSH connected)

```
sudo apt update && sudo apt upgrade -y
```
```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```
```
node -v && npm -v
```
```
sudo npm install -g pm2
```
```
sudo apt install -y nginx
```
```
cd ~
git clone <your github clone url>
cd <your directory>
```
```
npm install
```

```
nano .env

# Then your one cmd pallet will appear, there just paste your env file via `ctrl+v` or `ctrl+shift+v`

# Then press: `ctrl + O` 
# Then press: `Enter`
# Then press: `ctrl + X`
```

```
npm run build
```

```
pm2 start dist/server.js --name backend  
# your root file, and the name for your server which you want to give

pm2 save
pm2 startup
```
```
sudo truncate -s 0 /etc/nginx/sites-available/default
```
```
sudo nano /etc/nginx/sites-available/default
```

#### Then paste this there via `ctrl+v` or `ctrl+shift+v`
```
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:3000;  # backend PORT number
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```
sudo nginx -t
```

```
sudo systemctl restart nginx
```
```
sudo systemctl enable nginx
```

🎉 Hurre!!!! krdia deploye tumne apna backend `AWS` pe.
#### Test karo browser me: http://98.92.123.244 `your public ip`

<br><br>
`Now start setup your CICD flow:`
## STEP 3: GitHub Repository - Create Deploy Key
In EC2 Terminal:
```
ssh-keygen -t ed25519 -C "github-deploy-key" -f ~/.ssh/github_deploy_key -N ""
```

```
cat ~/.ssh/github_deploy_key.pub
```
Copy the complete row output (starts with ssh-ed25519)

### In Github: 
1. Go to repo > Settings > Deploy keys
2. Click `Add deploy key`
3. Title: `EC2 Deploy Key`
4. Paste the key
5. ✅ Check `Allow write access`
6. Click `Add key`

<br><br>

## STEP 4: GitHub Repository - Add Secrets
`Go to repo > Settings > secrets > actions`

Add these secrets (click "New repository secret" for each):
| Name     | Value    |
|----------|----------|
| EC2_HOST  | 98.92.123.244  #public ip address|
| EC2_USERNAME  | ubuntu  |
| EC2_SSH_KEY  | Paste contents of your .pem file |


<br><br>

## STEP 5: GitHub Repository - Create Workflow File
`Go to your repo`

1. Click "Add file" → "Create new file"
2. File path: .github/workflows/deploy.yml
3. Paste this:

```
name: Deploy to EC2

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd ~/<repo-name>
            git pull origin main
            npm install
            npm run build
            pm2 restart <your-server-name>
```
4. Click "Commit changes"


<br><br>

## STEP 6: AWS Console - Update Security Group

1. Go to EC2 → Instances → Click your instance
2. Click "Security" tab
3. Click the security group link
4. Click "Edit inbound rules"
5. Make sure you have:

| Type       | Port | Source        |
|------------|------|---------------|
| SSH        | 22   | Your IP or 0.0.0.0/0 |
| HTTP       | 80   | 0.0.0.0/0     |
| Custom TCP | 3000 #backend port | 0.0.0.0/0     |

6. Click "Save rules"

<br><br><br><br>

---
# ⚠️Quick Debug Commands:
### EC2 Terminal - Check karo backend chal raha hai ya nahi:
```
pm2 status
```

If error aaye, check logs:
```
pm2 logs <your-server-name>
```
---



<br><br><br><br><br><br>




---
Ho gaya Bhai! Ab koi bhi change push karo main branch me, automatic deploy ho jayega! 🎉