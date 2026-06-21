# 07-Cart Service

> **Hint** Cart Service manages shopping cart sessions. Built with Node.js, it uses Redis to store cart data with a 1-hour TTL. It calls the Catalogue Service to fetch product details.

> **Dependency** Redis and Catalogue Service must be running before starting Cart.

---

## Step 1: Install Node.js 20

```shell
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
dnf install -y nodejs
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
curl -L -o /tmp/cart.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/cart.zip
cd /app
unzip /tmp/cart.zip
npm install --production
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 4: Configure Systemd Service

Create the file `/etc/systemd/system/cart.service`:

```ini
[Unit]
Description=RoboShop Cart Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=10
SyslogIdentifier=cart

Environment=REDIS_HOST=localhost
Environment=CATALOGUE_URL=http://localhost:8002
Environment=PORT=8003

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` in `REDIS_HOST` with the private IP of the Redis server. Replace `localhost` in `CATALOGUE_URL` with the private IP of the Catalogue server.

---

## Step 5: Start and Verify

Enable and start the service:

```shell
systemctl daemon-reload
systemctl enable cart
systemctl start cart
```

Verify the service is running:

```shell
systemctl status cart
journalctl -u cart -f 
```
