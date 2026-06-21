# 12-Orders Service

> **Hint** Orders Service listens to `order.queue` on RabbitMQ, creates order records in MongoDB, calculates shipping costs by calling Shipping Service, and triggers Notification Service. Built with Java 17 + Spring Boot.

> **Dependency** MongoDB, RabbitMQ, Shipping Service, and Notification Service must be running before starting Orders.

---

## Step 1 — Install Java 21

```shell
dnf install -y java-21-openjdk java-21-openjdk-devel maven
```

---

## Step 2 — Add User, Download, Build, and Deploy

```shell
useradd -r -s /bin/false appuser
mkdir -p /app
curl -L -o /tmp/orders.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/orders.zip
mkdir -p /app && cd /app
unzip /tmp/orders.zip
mvn clean package -DskipTests
cp target/orders.jar /app/orders.jar
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 3 — Configure Systemd Service

Create `/etc/systemd/system/orders.service`:

```ini
[Unit]
Description=RoboShop Orders Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=java -jar /app/orders.jar
Restart=on-failure
RestartSec=10
SyslogIdentifier=orders

Environment=MONGO_URL=mongodb://localhost:27017/orders
Environment=AMQP_HOST=localhost
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=RoboShop@1
Environment=SHIPPING_URL=http://localhost:8004
Environment=NOTIFICATION_URL=http://localhost:8008
Environment=PORT=8007

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` values with the private IPs of MongoDB, RabbitMQ, Shipping, and Notification servers respectively.

---

## Step 4 — Enable and Start

```shell
systemctl daemon-reload
systemctl enable orders
systemctl start orders
```

---

## Step 5 — Verify

```shell
systemctl status orders
journalctl -u orders -f
```


