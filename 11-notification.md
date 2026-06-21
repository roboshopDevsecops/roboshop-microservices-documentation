# 11-Notification Service

> **Hint** Notification Service sends order confirmation emails. Built with Python 3 / Flask. It listens for events from Orders Service and sends emails via Brevo API (free tier: 300 emails/day). In development mode (no API key set), it logs emails to stdout instead.

> **Dependency** RabbitMQ must be running. Orders Service calls this service's REST endpoint after order creation.

---

## Step 1 — Install Python 3

```shell
dnf install -y python3 python3-pip
```

---

## Step 2 — Add User and Deploy

```shell
useradd -d /app -r -s /bin/false appuser
mkdir -p /app
curl -L -o /tmp/notification.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/notification.zip
cd /app
unzip /tmp/notification.zip
pip3 install -r requirements.txt
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 3 — Configure Systemd Service

Create `/etc/systemd/system/notification.service`:

```ini
[Unit]
Description=RoboShop Notification Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=/usr/local/bin/gunicorn -b 0.0.0.0:8008 app:app
Restart=on-failure
RestartSec=10
SyslogIdentifier=notification

Environment=BREVO_API_KEY=
Environment=FROM_EMAIL=noreply@roboshop.demo
Environment=FROM_NAME=RoboShop
Environment=PORT=8008

[Install]
WantedBy=multi-user.target
```

> **Note** Leave `BREVO_API_KEY` empty for development/demo — emails will be logged to stdout. Set it to a real Brevo API key for actual email delivery.

---

## Step 4 — Enable and Start

```shell
systemctl daemon-reload
systemctl enable notification
systemctl start notification
```

---

## Step 5 — Verify

```shell
systemctl status notification
journalctl -u notification -f
```

> **Re-deployment Note** If you redeploy the application zip, re-run the `pip3 install -r requirements.txt` step, then run `systemctl restart notification`.
