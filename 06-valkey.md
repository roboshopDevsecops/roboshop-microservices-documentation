# 06-Valkey (Redis Replacement)

> **RHEL 10 Note** RHEL 10 replaced Redis with Valkey, a fully compatible Redis fork. The Cart service works with Valkey without any code changes since Valkey uses the same protocol and default port (6379).

> **Hint** Valkey is used by the Cart service to store shopping cart sessions as key-value pairs with a TTL (time-to-live). When a session expires or a user checks out, Valkey automatically removes the entry.

---

## Install

Install Valkey from the default RHEL 10 AppStream repository and enable it on boot:

```shell
dnf install -y valkey
systemctl enable valkey
systemctl start valkey
```

---

## Configure

By default Valkey binds only to `127.0.0.1`. Because the Cart Service runs on a separate server, Valkey must accept remote connections.

Edit `/etc/valkey/valkey.conf` and change the following directives:

```
bind 0.0.0.0
protected-mode no
```

> **Important** Binding to `0.0.0.0` exposes Valkey on all interfaces. Valkey has no authentication enabled by default. Restrict access to port 6379 using firewall rules or a VPC security group so that only the Cart Service server can connect.

Restart the service to apply the change:

```shell
systemctl restart valkey
```

