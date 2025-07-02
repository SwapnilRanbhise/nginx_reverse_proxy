
# 🗂️ NGINX Reverse Proxy — Local Setup Notes

## ✅ Goal

- Test reverse proxy on my **personal laptop**.
- Clean up a long backend URL:
  ```
  https://supplier.unionsystechnologies.com:8443/ords/r/ust/unionsys-ats/login
  ```
  → Make it accessible locally as:
  ```
  http://localhost/login
  ```
- Serve my own `index.html` at `/`.
- Proxy only the **login page** and its **assets**.

---

## ⚙️ 1️⃣ Install NGINX (Ubuntu / Debian)

```bash
sudo apt update
sudo apt install nginx
```

---

## 📌 2️⃣ NGINX Default Behavior

- Default config: `/etc/nginx/sites-available/default`
- Serves files from:
  ```
  root /var/www/html;
  index index.html index.nginx-debian.html;
  ```
- Default index page: `index.nginx-debian.html`
- `http://localhost` → shows default welcome page.

---

## 🗂️ 3️⃣ Default Proxy Config (Cleaned Version)

Here’s my working **final config** for testing the login page with partial proxying:

```nginx
server {
    listen 80;
    server_name localhost;

    # Local static site
    root /var/www/html;
    index index.html;

    # Serve local files at "/"
    location / {
        try_files $uri $uri/ =404;
    }

    # Proxy the login page
    location /login {
        proxy_pass https://supplier.unionsystechnologies.com:8443/ords/r/ust/unionsys-ats/login/;
        proxy_set_header Host supplier.unionsystechnologies.com;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Proxy backend assets needed by login page
    location /ords/ {
        proxy_pass https://supplier.unionsystechnologies.com:8443/ords/;
        proxy_set_header Host supplier.unionsystechnologies.com;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🔑 4️⃣ How the Flow Works

**Request flow:**

```
Browser → localhost:80 (NGINX) → backend server
   |
   ├─ `/` → served locally (index.html)
   ├─ `/login` → proxied to backend login URL
   ├─ `/ords/` → proxied for backend CSS/JS/images
   ├─ any other path → 404 or served locally if file exists
```

- **If the backend uses other folders** (e.g., `/images/` or `/js/`), they must also be proxied for styles to work 100%.

---

## 🧩 5️⃣ Useful Commands

| Task | Command |
|------|---------|
| Test config syntax | `sudo nginx -t` |
| Start NGINX | `sudo systemctl start nginx` |
| Stop NGINX | `sudo systemctl stop nginx` |
| Reload config | `sudo nginx -s reload` or `sudo systemctl reload nginx` |
| Check status | `sudo systemctl status nginx` |
| Check errors | `sudo tail -f /var/log/nginx/error.log` |
| See config | `cat /etc/nginx/sites-available/default` |

---

## ⚠️ 6️⃣ Why It’s Not 100% Perfect

- The login page works because `/login` is proxied.
- The CSS/JS/images work because `/ords/` is proxied.
- If the backend needs other folders (`/images/`, `/favicon.ico`), those need to be proxied too.
- **Otherwise:** styling may look broken or incomplete.

---

## 🔒 7️⃣ About Port 80 (HTTP) vs Port 443 (HTTPS)

| HTTP (Port 80) | HTTPS (Port 443) |
|----------------|------------------|
| No encryption | Encrypted (TLS) |
| Good for local tests | Needed for real user data |
| Not safe for login forms | Safe for login forms |

- **Local test → OK to use HTTP.**
- **Production → MUST use HTTPS.**

---

## 🗝️ 8️⃣ How to Add HTTPS (Optional, Local)

- Generate self-signed certs:
  ```bash
  sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048       -keyout /etc/ssl/private/nginx-selfsigned.key       -out /etc/ssl/certs/nginx-selfsigned.crt
  ```
- Update config:
  ```nginx
  server {
      listen 443 ssl;
      server_name localhost;

      ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
      ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

      ...
  }
  ```

- Restart NGINX:
  ```bash
  sudo nginx -t
  sudo systemctl reload nginx
  ```

- Visit:
  ```
  https://localhost/login
  ```

---

## 🎯 Summary

✅ **What I built:**  
A working **local reverse proxy** that:
- Serves my own static site at `/`
- Proxies the backend login page at `/login`
- Proxies assets under `/ords/` so the login works with styles
- Uses `proxy_set_header` to fix **SNI / Host header** issues
- Keeps the backend partially hidden (safer than full proxy)
- Runs on **HTTP (port 80)** for test → switch to HTTPS for real use

---

**📍 Next Steps:**  
- Test more asset paths (`/images/`, `/js/`) if needed.
- Add HTTPS if moving to production.
- Optionally restrict who can access `/login`.

---

**✅ This is my full local flow — ready to explain!**
