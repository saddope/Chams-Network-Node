# Chasm Node Setup Guide

## Project Information
- **Twitter:** [@ChasmNetwork](https://twitter.com/ChasmNetwork)
- **Website:** [Visit](https://chasm.net)

## Node Information
**This node is incentivized.** Please ensure you meet the minimum requirements before proceeding with the setup.

### Minimum Requirements
- **Operating System:** Linux (linux/amd64, linux/arm64)
- **vCPU:** 1
- **RAM:** 1GB
- **Disk Space:** 20GB
- **IP Address:** Static IP

### Recommended Server Requirements
- **Operating System:** Ubuntu (20.04 LTS onwards)
- **vCPU:** 2
- **RAM:** 4GB
- **Disk Space:** 50GB SSD
- **IP Address:** Static IP

## Prerequisites

1. **Obtain API Key:**
   - Visit [Groq Console](https://console.groq.com/keys) and sign in.
   - Copy the API key and save it securely.

2. **Mint SCOUT_UID and WEBHOOK_API_KEY:**
   - Visit [Chasm Scout Mint](https://scout.chasm.net/private-mint) and mint your node with 0.025 $MNT gas fee.
   - After minting, click on the _mint(scout) button and copy the SCOUT_UID and WEBHOOK_API_KEY. Save them securely.

3. **Setup Ngrok:**
   - Sign up at [Ngrok](https://dashboard.ngrok.com/signup).
   - Go to the "Your Authtoken" section, reveal the authtoken, and save it securely.

## Installation

### 1. Update and Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo apt install screen -y
```

### 2. Install Ngrok
```bash
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update && sudo apt install ngrok
```

### 3. Configure Ngrok
Run the command copied from the Ngrok website (replace `yourauthtoken` with your actual authtoken):
```bash
ngrok config add-authtoken yourauthtoken
```

### 4. Start Ngrok
```bash
screen ngrok http 3001
```
- Copy the forwarding URL (e.g., `https://1bXX.XX.XXX.XXX.ngrok-free.app`) and save it. 
- Detach from the screen session by pressing `Ctrl + A + D`.

### 5. Create and Configure Environment File
Create a `.env` file and paste the following:
```env
PORT=3001
LOGGER_LEVEL=debug

ORCHESTRATOR_URL=https://orchestrator.chasm.net
SCOUT_NAME=tukangturu
SCOUT_UID=PASTE_UID_HERE
WEBHOOK_API_KEY=PASTE_API_KEY_HERE
WEBHOOK_URL=PASTE_NGROK_FREE_APP_URL_HERE
PROVIDERS=groq
MODEL=gemma2-9b-it
GROQ_API_KEY=PASTE_GROQ_API_KEY_HERE
```
Replace the placeholders (`PASTE_UID_HERE`, `PASTE_API_KEY_HERE`, etc.) with the actual values obtained earlier.

### 6. Configure Firewall
```bash
sudo ufw allow 3001
```

### 7. Deploy Docker Container
```bash
docker pull chasmtech/chasm-scout:latest
docker run -d --restart=always --env-file ./.env -p 3001:3001 --name scout chasmtech/chasm-scout
```

### 8. Verify Node Status
Check the logs to ensure the node is running correctly:
```bash
docker logs scout
```

Test the endpoint:
```bash
curl localhost:3001
```
Expected output: `OK`

### 9. Send Test Request
```bash
source ./.env
curl -X POST \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $WEBHOOK_API_KEY" \
     -d '{"body":"{\"model\":\"gemma2-9b-it\",\"messages\":[{\"role\":\"system\",\"content\":\"You are a helpful assistant.\"}]}"}' \
     $WEBHOOK_URL
```

## Node Status
You can check your node status [here](https://chasm.net/status). If you see a yellow or green status, your node is functioning properly. The yellow dot will turn green within 1 or 2 hours.
