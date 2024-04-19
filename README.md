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

MasOc homebrew
```bash
ls -l /opt/homebrew/etc/nginx/
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

4. Error log
MacOs Homebrew
```bash
cat /opt/homebrew/var/log/nginx/error.log
```

Ubuntu
```bash
cat /var/log/nginx/error.log
```

5. Configuration file structure
Nginx consists of modules which are controlled by directives specified in the configuration file. Directives are divided into simple directives and block directives. A simple directive consists of the name and parameters separated by spaces and ends with a semicolon (;). A block directive has the same structure as a simple directive, but instead of the semicolon it ends with a set of additional instructions surrounded by braces ({ and }).
It block directive can have other directives inside it, called a context.
Example of a simple directive:
- [event](https://nginx.org/en/docs/ngx_core_module.html#events)
- [http](https://nginx.org/en/docs/http/ngx_http_core_module.html#http)
- [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server)
- [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)

Directives placed in the configuration file outside of any contexts are considered to be in the [main](https://nginx.org/en/docs/ngx_core_module.html) context. The events and http directives reside in the main context, server in http, and location in server.

6. Serve a static content
   1. First, create a directory `/data/www` and create an `index.html` file.
    ```
    mkdir -p /data/www
    echo "Hello, NGINX" > /data/www/index.html
    ```
    2. Second, create a directory `/data/images` and put some images in it.
    ```bash
    mkdir -p /data/images
    ```
    3. Next, Open the configuration file and add the following configuration.
    ```bash
     vim /opt/homebrew/etc/nginx/nginx.conf (your path may be different)
    ```
   4. Add the following configuration to the `http` context.
    Generally, the configuration file may include several server blocks [distinguished](https://nginx.org/en/docs/http/request_processing.html) by ports on which they [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) to and by [server names](https://nginx.org/en/docs/http/server_names.html). 
    Once nginx decides which server processes a request, 
    it tests the URI specified in the request’s header against the parameters of the location directives defined inside the server block.
    ```
    http {
        server {
            location / {
                root /data/www;
            }
        }
    }
    ```
    This `location` block specifies the “/” prefix compared with the URI from the request. 
    For matching requests, the URI will be added to the path specified in the [root](https://nginx.org/en/docs/http/ngx_http_core_module.html#root) directive, that is, to `/data/www`, to form the path to the requested file on the local file system. If there are several matching location blocks nginx selects the one with the longest prefix. The location block above provides the shortest prefix, of length one, and so only if all other location blocks fail to provide a match, this block will be used.
    Add another `location` block to serve images.
    ```
    location /images/ {
        root /data;
    }
    ```
    
    It will be a match for requests starting with `/images/` (location / also matches such requests, but has shorter prefix).
    This is already a working configuration of a server that listens on the standard port 80 and is accessible on the local machine at http://localhost/.
    o apply the new configuration, you need to reload nginx.
    ```bash
    nginx -s reload
    ```
