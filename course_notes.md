# Udemy NGINX Beginner's Guide

# Section 2: Installation
## Lesson 5
### NGINX set-up via `apt`
Update apt definitions via `apt get update`. Get and install ngin with `apt-get install nginx`. This also loads NGINX as well.

### Check processes
See active processes by `ps -aux | grep nginx`

### Starting NGINX
Can use Ubuntu's `service` function by `service nginx start`. 

## Lesson 6
### update package definitions
apt-get update

### get and extract tarball of NGINX server
wget <uri from nginx.org>
tar -zxvf <tarball object>
cd <new directory>

### NGINX server configuration and compiler settings
apt-get install build-essential
./configure
apt-get install <packages needed exposed by ./configure error>
### example used in the lecture for needed for this install
apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
### example used in the lecture using some basic best practice locations for the configuration
./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
make
make install


## Lesson 7
 - `systemd` service will manage start, stop, restart
 - `systemctl` == `systemd` command
 - Use `init.d` from <nginx.com> to manage automation of on-start, restart processes

### From <https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/>
### From <https://www.nginx.com/resources/wiki/start/topics/examples/systemd/>
Description=The NGINX HTTP and reverse proxy server
```
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

 - `systemctl enable nginx` is a one-off configuration change to start nginx on system start
```
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
```

## Lesson 8
 - NGINX is native for Linux, not intended to be used on Windows machines.
 - So... don't use it on Windows.
 
## Lesson 9: Understanding Configuration Terms
 - two main terms:
   - context	: scope, sections within a configuration. Nested, inherit from parents. Configuration file is the `Main` context
     - Main		: the global context, everything in a configuration
     - http		:
     - server	: similar to a vhost
     - location	: match URI locations to incoming requests to the parent server context
   - directive	: specific configuration options set in configuration files
     - name value;

# Secion 3: Configuration
## Lesson 9: Configuration Terms
### Context and Directive
 - Directives are specific options set within a Context `name value;`
 - Contexts are sections within the configuration wherein a Directive can be set
   - Very much like scope
   - Contexts can nest and inherit from parents
   - Examples: http > server > location

## Lesson 10: Creating a Virtual Host
### /etc/nginx/nginx.conf
 - Configuration for the NGINX server
 - Use `http` context to specify how website is served
   - Use `server` context to configure where server listens
   - Use `types` context to configure how filetypes are handled

### Browser dev mode
 - Use this to see how site is being served
 - <Shift Command-R> to reload page without cache

## Lesson 11: Location Blocks
### Location Blocks are contexts in the nginx.conf
 - Each location is associated with a URI
 - Locate them within the `http/server` context
   - Preferential Prefix Matching: ```location ^~ /uri {}``` 
   - Prefix Matching: ```location /uri {}``` 
   - Exact Matching: ```location = /uri {}``` 
   - Regex Matching: ```location ~ /regex {}``` 
   - Regex Matching (case insensitive): ```location ~* /regex {}``` 
 - Order of URI matching preference: Exact > Preferential Prefix > Regex > Prefix
 
## Lesson 12: Variables
### Variables
#### Configuration Variables
 - Variables that are user set
 - `set $foo 'bar'`

#### NGINX
 - `$http, $uri, $args` see [the docs](http://nginx.org/en/docs/varindex.html)
 
### Conditionals
 - `if ( logic ) {}`

### URI supplied variables
 - ?foo1=bar1&foo2=bar2...
 - regex groups unpacked as $1, $2, etc...
 
## Lesson 13: Rewrites and Redirects
### Rewrites
#### `rewrite` directive
 - literally rewrites the submitted URI and reruns it through the engine
 - use `last` flag to prevent multiple rewrites

#### `return` directive
 - returns a status code and some data
 - a 3xx allows the data to be a new URI
 
## Lesson 14: Try-Files & Named Locations
 - rewrites only on last try options

## Lesson 15: Logs
 - we configured the logging location during build from source
 - can add logging per location in the configuration `access_log`
   - toggle this to `off` to reduce noise
   
## Lesson 16: Inheritance and Directive Types
### Directive Types
 - Standard
 - Array
 - Action
 
## Lesson 17: PHP
 - NGINX does not natively run PHP or other processes
 - use another service to run PHP and communicate with NGINX

## Lesson 18: Worker Processes
### Master and Worker
 - master process is the NGINX process that `systemctl` starts
 - worker process spawned by master process to handle requests
   - defaults to 1 worker
	 - `worker_processes` in the main context controls this
	 - `worker_processes auto` sets to number of cores available

### Performance
 - number of workers should be limited to machine capabilities
 - get number of cores on computer via nproc
 - `events>worker_connections` is the first events directive we consider
   - sets number of connections each worker can accept
	 - see available limit via `ulimit -n`
 - get maximum number of concurrent requests by worker_processes x worker_connections

## Lesson 19: Buffers and Timeouts
 - these are dependent on the nature of the requests to a server, not the physical availability of the server
 - data in: http request > tcp > NGINX > RAM | disk 
 - buffer is the RAM used by a worker for requests in process
 - timeout is the time alloted for a job to complete before killing the process
 - some important configurables
   - `sendfile on|off;` controls if buffers should be used for static resources
	 - `tcp_nopush on|off;` optimizes the size of the packets to send over tcp
 - controlling these options may not be encouraged without knowledge of how clients are using the server

## Lesson 20: Adding Dynamic Modules
### Static Modules
 - what we installed during the setup of the server
 - these are persistent and available (always loaded)

### Dynamic Modules
 - installed like static modules
 - not automatically included by the NGINX server
 - controlled by the configuration of the server

### Installing
 - we will rebuild from source, including more modules
 - either rebuild from existing source, or new (this is essentially how to upgrade)
 - what was the original build?
```
$ nginx -V
nginx version: nginx/1.19.0
built by gcc 9.3.0 (Ubuntu 9.3.0-10ubuntu2)
built with OpenSSL 1.1.1f  31 Mar 2020
TLS SNI support enabled
configure arguments: --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
```
 - `--modules-path` flag controls location of module(s) installed
 - use `load_module` directive with path to source to load the module in the configuration

# Section 4: Performance
 - covers useful modules not included in base NGINX

## Lesson 21: Headers & Expires
 - Expires headers are a response header that inform a client or browser how long data will be cached for
 - Use general location tagging < ~* \.(.css|.jpg ...)$ > ~*to control expires headers!

## Lesson 22: Compressed Responses
 - when a client requests a resource, client can request a compressed response
 - client handles compression | decompression
 - control this in the `http` context using `gzip on|off;` directive
 - controlled by a parameter `gzip_comp_level` from [0-9]
   - after 5, compression results do not get much smaller (logarithmic tradeoff)
	 - suggested values of 3-4 for maximum benefits with minimal server performance tradeoff
 - array type directive `gzip_types` controls what kinds of files are compressed
 - use the header `Vary Accept-Encoding` to allow client to flag whether compression is allowed or not
   - test with `curl -I -H "Accept-Encoding: gzip, deflate" http://159.89.82.8/style.css`

## Lesson 23: FastCGI Cache
 - a Micro Cache that stores server side that handles dynamic language responses to avoid server side language processing
   - useful with PHP and mySQL queries for example
 - Test using Apache Bench `apt-get install apache2-utils`
   - use `ab -n number_of_requests -c number_concurrent_requests http://...`

## Lessons 24|25: HTTPS2 and HTTPS2 Pushing
 - HTTPS2 is transmitted in binary and should be preferred for speed
 - Pushing is allowed in HTTPS2, which lets the server push resources when a particular resource is fetched

# Section 5: Security
## Lesson 26: HTTPS (SSL)
 - TSL (supercedes SSL) is the preferred communication protocol
 - expected that https is used instead of http
   - simply create another server context to listen on port 80, redirect to 443
 - SSL is the defacto method by NGINX
 - specify the use of TSL and name the acceptable ciphers
```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;
ssl_dhparam /etc/nginx/ssl/dhparam.pem;
```
 - cipher lists should be sought out online and note they WILL become outdated
 - Use Diffie-Hellman cipher security methods <https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange>
```
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

## Lesson 27: Rate-Limiting
 - use [Siege](https://github.com/JoeDog/siege) to load test sites
 - use rate to limit by rate
 - use burst to allow a bundle of requests at a time
 - use nodelay with burst to go faster than specified rate

## Lesson 28: Basic Auth
 - possible to create credentials for areas of the site where admins might need to work in

## Lesson 29: Hardening NGINX
 - `servertokens on|off;` controls if header containing NGINX version is transmitted (turn it off!)
 - `server> add_header X-Frame-Options "SAMEORIGIN"` to prevent embedding your page
 - `server> add_header X-Xss-Protection "1; mode=block";` to prevent cross-site scripting
 -  remove default modules that are not used (find defaults with the `without` flag via configure)

## Lesson 30: Let's Encrypt SSL Certificates
 - use certbot to manage SSL certificates
 - use cron jobs to automate renewing the certificates
 - certbot will generate the required configuration settings in the nginx.conf file

# Section 6: Reverse Proxy and Load Balancing
## Prerequisites
 - Use PHP as a dummy server for testing

## Lesson 31: Reverse Proxy
 - masks client from the server
 - handles requests from the client and acts as the source of truth
 - then manages the request and talks to the services requested
 - `proxy_pass` directive controls the proxy
 - `add_header proxied nginx` sends the proxy header
 - `proxy_set_header proxied nginx` sends the proxy header via the proxy

## Lesson 32: Load Balancer
 - use the `upstream proxy_name` context to designate the servers for balancing
 - use `proxy_pass http://proxy_name` to send to the balancer
 - test with bash while loop `while sleep 0.5; do curl http://localhost:80; done`

## Lesson 33: Load Balancer Options
### Sticky Sessions [ip hash]
 - once client has connected to server, keep requests to the first server it was directed to
 - configured by placing the `ip_hash` directive in the `http>upstream` context

### Active Connections [session state]
 - proxies to the server with the least connections
 - use `least_conn` directive in the `http>upstream` context

