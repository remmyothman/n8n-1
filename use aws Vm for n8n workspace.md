# Setting Up an Amazon EC2 Instance for Your Personal n8n Workspace

Follow these steps to configure an Amazon EC2 instance for running n8n:

## Step 1: Update Your System
Ensure your system is up-to-date:

```bash
sudo yum update -y
```

## Step 2: Install Node.js and npm
n8n requires Node.js to run. Install the LTS version of Node.js:

```bash
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo yum install -y nsolid
```

Verify the installation of Node.js and npm:

```bash
node -v
npm -v
```

## Step 3: Install n8n
Install n8n globally:

```bash
sudo npm install -g n8n
```

Verify the installation:

```bash
n8n --version
```

## Step 4: Run n8n
Start n8n:

```bash
n8n
```

By default, it will run on port `5678`. Access it in your browser at:

```
http://<Instance_Public_IP>:5678
```

## Go to Aws instance security group and add the inbound rule for port 5678.

## Step 5: Set Up n8n as a Background Service
Running n8n in the foreground is not ideal. Configure it as a background service:

### Create a Systemd Service File

```bash
sudo nano /etc/systemd/system/n8n.service
```

Add the following content, replacing `<your-user>` with your Linux username:

```ini
[Unit]
Description=n8n workflow automation tool
After=network.target

[Service]
ExecStart=/usr/bin/n8n
Restart=always
User=<your-user>
Environment=HOME=/home/<your-user>
Environment=DATA_FOLDER=/home/<your-user>/.n8n

[Install]
WantedBy=multi-user.target
```

### Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
```

Check the service status:

```bash
sudo systemctl status n8n
```

If you are unable to connect to your n8n workspace, proceed with the next steps.

## Step 6: Install OpenSSL for HTTPS
Install OpenSSL and generate SSL certificates:

```bash
sudo yum install openssl -y
sudo mkdir /etc/n8n-certs
sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/n8n-certs/n8n-key.pem -out /etc/n8n-certs/n8n-cert.pem -days 365 -nodes
```

Verify the generated files:

```bash
ls /etc/n8n-certs
```

To check file permissions, use:

```bash
ls -l /etc/n8n-certs
```

Grant access to the EC2 user for these files:

```bash
sudo chown ec2-user:ec2-user /etc/n8n-certs/n8n-key.pem
sudo chown ec2-user:ec2-user /etc/n8n-certs/n8n-cert.pem
```

## Step 7: Update the Service File for HTTPS
Modify the service file:

```bash
sudo nano /etc/systemd/system/n8n.service
```

Replace the content with the following, updating `your_public_ip_address`, `your_username`, and `your_password`:

```ini
[Unit]
Description=n8n service
After=network.target

[Service]
Type=simple
User=ec2-user
ExecStart=/usr/bin/npm run start
Environment=N8N_HOST=your_public_ip_address
Environment=N8N_PORT=5678
Environment=N8N_PROTOCOL=https
Environment=N8N_BASIC_AUTH_ACTIVE=true
Environment=N8N_BASIC_AUTH_USER=your_username
Environment=N8N_BASIC_AUTH_PASSWORD=your_password
Environment=N8N_SSL_KEY=/etc/n8n-certs/n8n-key.pem
Environment=N8N_SSL_CERT=/etc/n8n-certs/n8n-cert.pem
Restart=always

[Install]
WantedBy=multi-user.target
```

### Restart the Service

```bash
sudo systemctl daemon-reload
sudo systemctl restart n8n
```

Your n8n instance should now be running securely with HTTPS.
