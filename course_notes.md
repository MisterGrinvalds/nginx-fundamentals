# Udemy NGINX Beginner's Guide

# Section 2: Installation
## Lesson 6
### update package definitions
apt-get update

### get and extract tarball of nginx server
wget <uri from nginx.org>
tar -zxvf <tarball object>
cd <new directory>

### nginx server configuration and compiler settings
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
```Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.```

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
## Lesson 10: Creating a Virtual Host
