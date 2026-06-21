# 10-Payment Service

> **Hint** Payment Service processes checkout and publishes order events to RabbitMQ. Built with Python 3 / FastAPI. It validates cart and user data, then puts a message on the `order.queue` for Orders Service to consume.

> **Dependency** RabbitMQ, Cart Service, and User Service must be running before starting Payment.

---

## Step 1 — Install Python 3

```shell
dnf install -y python3 python3-pip
python3 --version
```

---

## Step 2 — Add Application User and Directory

```shell
useradd -r -s /bin/false appuser
mkdir -p /app
```

---

## Step 3 — Download and Install

```shell
curl -L -o /tmp/payment.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/payment.zip
cd /app
unzip /tmp/payment.zip
pip3 install -r requirements.txt
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 4 — Configure Systemd Service

Create `/etc/systemd/system/payment.service`:

```ini
[Unit]
Description=RoboShop Payment Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=/usr/local/bin/uvicorn main:app --host 0.0.0.0 --port 8005
Restart=on-failure
RestartSec=10
SyslogIdentifier=payment

Environment=AMQP_HOST=localhost
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=RoboShop@1
Environment=CART_URL=http://localhost:8003
Environment=USER_URL=http://localhost:8001
Environment=PORT=8005

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` values with the private IPs of RabbitMQ, Cart, and User servers respectively.

---

## Step 5 — Enable and Start

```shell
systemctl daemon-reload
systemctl enable payment
systemctl start payment
```

---

## Step 6 — Verify

```shell
systemctl status payment
journalctl -u payment -f 
```

> **Re-deployment Note** If you redeploy the application zip, re-run the `pip3 install -r requirements.txt` step, then run `systemctl restart payment`.
