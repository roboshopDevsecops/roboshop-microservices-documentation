# 09-RabbitMQ

> **Hint** RabbitMQ is the message broker for the asynchronous order pipeline. The Payment service publishes order events to a queue, the Orders service consumes and persists them, and then triggers the Notification service to send confirmations.

---

## Install

### Install Erlang

RabbitMQ requires Erlang. Use the official RabbitMQ team's packagecloud repository to ensure a compatible version:

```shell
cat > /etc/yum.repos.d/rabbitmq_erlang.repo << 'EOF'
[rabbitmq_erlang]
name=rabbitmq_erlang
baseurl=https://packagecloud.io/rabbitmq/erlang/el/9/$basearch
gpgcheck=0
enabled=1
EOF
dnf install -y erlang
```

### Add the RabbitMQ Repository and Install

```shell
cat > /etc/yum.repos.d/rabbitmq_rabbitmq-server.repo << 'EOF'
[rabbitmq_rabbitmq-server]
name=rabbitmq_rabbitmq-server
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/9/$basearch
gpgcheck=0
enabled=1
EOF
dnf install -y rabbitmq-server
```

> **RHEL 10 Note** The packagecloud setup scripts do not support RHEL 10 yet. Manual repo files using the el/9 path are used instead, with GPG checking disabled.

> **Note** Using the official RabbitMQ packagecloud repositories ensures that the Erlang and RabbitMQ versions are compatible. Do not mix packages from the default RHEL AppStream repository with packages from packagecloud.

### Enable and Start

```shell
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
```

---

## Configure

### Enable the Management UI

The management plugin provides a browser-based UI for monitoring queues, exchanges, and connections. It is optional but strongly recommended during initial setup:

```shell
rabbitmq-plugins enable rabbitmq_management
```

Restart the service after enabling the plugin:

```shell
systemctl restart rabbitmq-server
```

> **Dependency** The Payment, Orders, and Notification services must all have the correct RabbitMQ hostname or IP set in their configuration files. Record the private IP of this server before proceeding.

### Default Credentials

The default `guest` user can only connect from `localhost`. For the application services to connect remotely, create a dedicated user:

```shell
rabbitmqctl add_user roboshop RoboShop@1
rabbitmqctl set_user_tags roboshop administrator
rabbitmqctl set_permissions -p / roboshop ".*" ".*" ".*"
```

> **Important** The `guest` account is intentionally restricted to localhost connections by RabbitMQ's default configuration. Always create a named application user for remote access rather than relaxing the guest restriction.

---

## Start

RabbitMQ is already started during the Install step. To restart manually if needed:

```shell
systemctl restart rabbitmq-server
```

---

## Verify

Check that the service is active:

```shell
systemctl status rabbitmq-server
```

Confirm the broker is running and display node information:

```shell
rabbitmqctl status
```

List the current users to confirm the `roboshop` user was created:

```shell
rabbitmqctl list_users
```

Access the management UI in a browser at:

```
http://<SERVER-IP>:15672
```

Log in with the `roboshop` / `RoboShop@1` credentials created above.

> **Note** Port 15672 is the management UI. Port 5672 is the AMQP port used by the application services. Both must be open in the server's security group.

> **Re-deployment Note** If you need to update RabbitMQ, run `dnf update -y rabbitmq-server erlang` and then `systemctl restart rabbitmq-server`. Queued messages that have been persisted to disk are retained across restarts. Transient (non-durable) messages will be lost.
