# 01-Frontend

> **Hint** The frontend is a Next.js application exported as static HTML/CSS/JS files, served by Nginx. Nginx also acts as an API gateway, routing `/api/*` paths to the appropriate backend services based on path prefix.

> **Dependency** All backend services should be running. The frontend will load but API calls will fail until backends are up.

---

## Step 1 — Install Nginx 1.26

```shell
dnf install -y nginx
systemctl enable nginx
systemctl start nginx
```

> **RHEL 10 Note** RHEL 10 dropped module streams. Nginx 1.26 is available directly from the AppStream repository.

---

## Step 2 — Install Node.js 20

```shell
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
dnf install -y nodejs
```

---

## Step 3 — Download, Build, and Deploy

```shell
curl -L -o /tmp/frontend.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/frontend.zip
mkdir -p /tmp/frontend && cd /tmp/frontend
unzip /tmp/frontend.zip
npm install
npm run build
rm -rf /usr/share/nginx/html/*
cp -r out/* /usr/share/nginx/html/
```

> **Hint** `npm run build` with `output: 'export'` in `next.config.js` generates static files in the `out/` directory. No Node.js server is needed at runtime.

---

## Step 4 — Configure Nginx

Replace the entire contents of `/etc/nginx/nginx.conf` with:

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;

        location /api/catalogue/ {
            rewrite ^/api/catalogue/(.*)$ /$1 break;
            proxy_pass http://localhost:8002;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/user/ {
            rewrite ^/api/user/(.*)$ /$1 break;
            proxy_pass http://localhost:8001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/cart/ {
            rewrite ^/api/(.*)$ /$1 break;
            proxy_pass http://localhost:8003;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/shipping/ {
            rewrite ^/api/(.*)$ /$1 break;
            proxy_pass http://localhost:8004;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/payment/ {
            rewrite ^/api/(.*)$ /$1 break;
            proxy_pass http://localhost:8005;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/ratings {
            rewrite ^/api/(.*)$ /$1 break;
            proxy_pass http://localhost:8006;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/orders/ {
            rewrite ^/api/(.*)$ /$1 break;
            proxy_pass http://localhost:8007;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location / {
            try_files $uri $uri/index.html $uri.html /index.html;
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
            expires 7d;
            add_header Cache-Control "public";
        }
    }
}
```

> **Important** All upstream addresses are set to `localhost` by default so Nginx starts without errors. As you set up each service on its own server, update the corresponding `localhost` to that server's private IP and run `systemctl restart nginx`. Use IP addresses (not hostnames) to avoid IPv6 resolution issues.

---

## Step 5 — Test and Restart Nginx

```shell
nginx -t
systemctl restart nginx
```

---

## Step 6 — Verify

Access `http://<FRONTEND-SERVER-PUBLIC-IP>` in your browser. The RoboShop storefront should load. Test by browsing products — they should appear if Catalogue Service is running.

> **Re-deployment Note** If you rebuild the frontend, re-run Steps 3 and 5 (`npm ci`, `npm run build`, copy files, then `nginx -t` and `systemctl restart nginx`). No systemd unit reload is needed since Nginx is already running.
