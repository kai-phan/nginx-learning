NGINX FOR BEGINNER

# What is NGINX?
NGINX is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

# Installation
1. MacOs
```bash
brew install nginx
```

2. Ubuntu
```bash
sudo apt update
sudo apt install nginx
```

# Location of configuration files
```bash
nginx -t
```
MacOs
```bash
ls -l /usr/local/etc/nginx/
```
Ubuntu
```bash
ls -l /etc/nginx/
```

1. Start the server
MacOs
```bash
brew services start nginx
```

Ubuntu
```bash
sudo systemctl start nginx
```

2. Check nginx process
```bash
ps aux | grep nginx
```

3. Access log
MacOs Homebrew
```bash
cat /opt/homebrew/var/log/nginx/access.log
```

Ubuntu
```bash
cat /var/log/nginx/access.log
```

For the first time cat `access.log` file, you may not see any logs. You need to access the server using the public IP address or domain name.

```bash
curl http://localhost:8080
cat /opt/homebrew/var/log/nginx/access.log
```

`127.0.0.1 - - [19/Apr/2024:07:45:44 +0700] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"`

The second time you access the server, you will see the log in the `access.log` file.

`127.0.0.1 - - [19/Apr/2024:08:01:25 +0700] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"`

Nginx cache the response, so the second time you access the server, the status code is `304` and the size is `0`.

