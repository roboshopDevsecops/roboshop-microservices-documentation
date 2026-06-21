# 05-User Service

> **Hint** User Service handles registration, login, and profile management. It is built with Node.js and stores data in MongoDB. It issues JWT tokens used by other services.

> **Dependency** MongoDB must be running and accessible before starting this service.

---

## Step 1: Install Node.js 20

```shell
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
dnf install -y nodejs
node --version
npm --version
```

---

## Step 2: Add Application User and Create Directory

```shell
useradd -r -s /bin/false appuser
mkdir -p /app
```

---

## Step 3: Download and Install

```shell
curl -L -o /tmp/user.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/user.zip
cd /app
unzip /tmp/user.zip
npm install --production
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 4: Configure Systemd Service

Create the file `/etc/systemd/system/user.service`:

```ini
[Unit]
Description=RoboShop User Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=10
SyslogIdentifier=user

Environment=MONGO_URL=mongodb://localhost:27017/users
Environment=JWT_SECRET=roboshop-secret-key
Environment=PORT=8001

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` in `MONGO_URL` with the private IP of the MongoDB server.

---

## Step 5: Start and Verify

Enable and start the service:

```shell
systemctl daemon-reload
systemctl enable user
systemctl start user
```

Verify the service is running:

```shell
systemctl status user
journalctl -u user -f
```
