# Connecting to an EC2 Instance and Setting Up Git Authentication and Set Up Backend and Frontend

## Step 1: Connect to EC2
```sh
 ssh -i "intellex-live.pem" ec2-user@3.24.144.123
```

## Step 2: Install Git
```sh
sudo yum install git -y
```

## Step 3: Install NVM (Node Version Manager)
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

## Step 4: Export NVM and Load Environment Variables
```sh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
source ~/.bashrc
```

## Step 5: Install Node.js via NVM
```sh
nvm install --lts
```

## Step 6: Authenticate Git on EC2 Using SSH Keys

### Generate an SSH Key
```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Add the SSH Key to the SSH Agent
```sh
eval "$(ssh-agent -s)"
```
```sh
ssh-add ~/.ssh/id_rsa
```

### Copy the Public Key to Your Clipboard
```sh
cat ~/.ssh/id_rsa.pub
```
- Copy the output.

### Add the SSH Key to Your Git Account
1. Go to your Github/GitLab profile.
2. Navigate to **Settings > SSH Keys**.
3. Paste the public key into the **"Key"** field and give it a title.
4. Click **Add key**.

### Check git user
```sh
ssh -T git@gitlab.com (Gitlab)
ssh -T git@github.com (Github)
```
- Copy the output.

## Step 7: Clone the Repository
```sh
git clone git@github.com:intellex-academy/Intellex-Academy-Core.git
```

## Step 8: Checkout to the Production Branch
```sh
git checkout main
```

## Step 8: Set up Backend
```sh
 cd Intellex-Academy-Core
 cd backend
 npm install
```

#### Create a `.env` File
- Add the necessary environment variables and their values.

#### Add pm2 and run node server
```sh
 npm install -g pm2
 pm2 start "src/app.js" --name intellex-backend-prod
 pm2 save
```

## Step 9: Set up Frontend
```sh
 cd Intellex-Academy-Core
 cd frontend
 npm install
 npm run build
```