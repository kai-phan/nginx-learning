NGINX FOR BEGINNER

# What is NGINX?
NGINX is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

# Installation
## MacOs
```bash
brew install nginx
```
## Ubuntu
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

## Start the server
MacOs
```bash
brew services start nginx
```
or 
```bash
brew services restart nginx
```

Ubuntu
```bash
sudo systemctl start nginx
```
## Check nginx process
```bash
ps aux | grep nginx
```
## Access log
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

## Error log
MacOs Homebrew
```bash
cat /opt/homebrew/var/log/nginx/error.log
```

Ubuntu
```bash
cat /var/log/nginx/error.log
```
## Configuration file structure
Nginx consists of modules which are controlled by directives specified in the configuration file. Directives are divided into simple directives and block directives. A simple directive consists of the name and parameters separated by spaces and ends with a semicolon (;). A block directive has the same structure as a simple directive, but instead of the semicolon it ends with a set of additional instructions surrounded by braces ({ and }).
It block directive can have other directives inside it, called a context.
Example of a simple directive:
- [event](https://nginx.org/en/docs/ngx_core_module.html#events)
- [http](https://nginx.org/en/docs/http/ngx_http_core_module.html#http)
- [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server)
- [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)

Directives placed in the configuration file outside of any contexts are considered to be in the [main](https://nginx.org/en/docs/ngx_core_module.html) context. The events and http directives reside in the main context, server in http, and location in server.

## Serve a static content
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

## Proxy server
One of the frequent uses of nginx is setting it up as a proxy server, which means a server that receives requests, passes them to the proxied servers, retrieves responses from them, and sends them to the clients.

1. First, define the proxied server by adding one more server block to the nginx’s configuration file with the following contents:
```bash
server {
    listen 8080;
    root /data/up1;
    
    location / {
      
    }
}
```

2. Next, use the server configuration from the previous section and modify it to make it a proxy server configuration. In the first location block, put the [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive with the protocol, name and port of the proxied server specified in the parameter (in our case, it is http://localhost:8080):
```bash
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```
We will modify the second `location` block, which currently maps requests with the `/images/` prefix to the files under the `/data/images` directory, to make it match the requests of images with typical file extensions. The modified location block looks like this:
```
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```
The parameter is a regular expression matching all URIs ending with `.gif`, `.jpg`, or `.png`. A regular expression should be preceded with ~. The corresponding requests will be mapped to the /data/images directory.
There are many [more](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) directives that may be used to further configure a proxy connection.