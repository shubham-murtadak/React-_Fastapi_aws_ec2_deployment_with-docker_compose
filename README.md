# React-_Fastapi_aws_ec2_deployment_with-docker_compose
This repo contains details instructions to deploy React frontend + Fastapi backend + with aws ec2 instance + docker compose


# Deployment Documentation

## 1. Create an EC2 Instance
- Navigate to your AWS console and create a new EC2 instance with the desired specifications.
- Make sure to choose an Ubuntu-based instance (e.g., `Ubuntu 20.04`).
- Allocate a new Elastic IP to ensure your IP address remains static.
  
## 2. Store Your SSH Key in a Specific Folder
- Download your SSH key (e.g., `my-react-key.pem`) and store it in a folder (e.g., `~/Downloads`).

## 3. Allocate Elastic IP Address
- Go to the **Elastic IP** section in the EC2 Dashboard.
- Allocate a new Elastic IP and associate it with your EC2 instance.

## 4. Configure Security Group for the EC2 Instance
- Navigate to **Security Groups** and select the security group for your EC2 instance.
- Add inbound rules to open ports for both frontend and backend services:
  - **Frontend (React)**: Port `3000` (TCP)
  - **Backend (FastAPI)**: Port `8000` (TCP)
  - Example:
    - `Port: 8000, Protocol: TCP, Source: 0.0.0.0/0` (for backend)
    - `Port: 3000, Protocol: TCP, Source: 0.0.0.0/0` (for frontend)

## 5. Navigate to the Folder Containing Your SSH Key
```bash
cd ~/Downloads
```

## 6. Copy Your Code to the EC2 Instance
Use `scp` (secure copy) to transfer your code to the EC2 instance:
```bash
scp -i "my-react-key.pem" -r "/path/to/Chatbot" ubuntu@<EC2_Public_IP>:/home/ubuntu/
```
Replace `/path/to/Chatbot` with the correct path to your project, and `<EC2_Public_IP>` with your EC2's public IP address.

## 7. Connect to Your EC2 Instance
To connect to your EC2 instance via SSH:
```bash
ssh -i "my-react-key.pem" ubuntu@<EC2_Public_IP>
```

## 8. Setup Dockerfile for Backend and Frontend

### React Frontend (Dockerfile)
```dockerfile
# Step 1: Use official Node.js image (v20.18.1)
FROM node:20.18.1-alpine as build

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install --legacy-peer-deps

# Copy the rest of the application files
COPY . .

# Build the React app
RUN npm run build

# Step 2: Serve the React app using a simple HTTP server
FROM node:20.18.1-alpine

# Install serve to serve the build folder
RUN npm install -g serve

# Set working directory
WORKDIR /app

# Copy the build directory from the previous step
COPY --from=build /app/build /app/build

# Expose port 3000
EXPOSE 3000

# Start the React app with serve
CMD ["serve", "-s", "build", "-l", "3000"]
```

### FastAPI Backend (Dockerfile)
```dockerfile
# Step 1: Use official Python 3.11.9 image
FROM python:3.11.9-slim

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy all code
COPY . /app

# Expose port 8000
EXPOSE 8000

# Run FastAPI
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 9. Create Docker Compose File
Create a `docker-compose.yml` file to set up both services (frontend and backend):
```yaml
version: '3'

services:
  frontend:
    build:
      context: ./frontend
    ports:
      - "3000:3000"
    networks:
      - app_network

  backend:
    build:
      context: ./backend
    ports:
      - "8000:8000"
    env_file:
      - ./backend/.env
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

## 10. Install Docker and Docker-Compose on EC2 Instance

### Docker Installation
```bash
sudo apt-get update
sudo apt-get install -y docker.io
```

### Docker Compose Installation
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r .tag_name)/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Verify Installations
```bash
docker --version
docker-compose --version
```

## 11. Build and Run the Application on EC2
Run the following command to build and start the services in the background:
```bash
sudo docker-compose up --build -d
```

## 12. Access the Application
- **Frontend (React)**: [http://<EC2_Public_IP>:3000/](http://<EC2_Public_IP>:3000/)
- **Backend (FastAPI)**: [http://<EC2_Public_IP>:8000/](http://<EC2_Public_IP>:8000/)

Make sure to replace `<EC2_Public_IP>` with your actual EC2 Elastic IP address.
