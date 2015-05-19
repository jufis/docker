#Quick guide on how to setup private docker registry over SSL

I'm assuming a fedora 20 linux 64bit box here.

First install docker-registry with yum:

>sudo yum install -y docker-registry

The install httpd,mod_ssl and openssl:

>sudo yum install -y httpd

>sudo yum install -y mod_ssl

>sudo yum install -y openssl


Create a ssl dir as root user:

>sudo mkdir /etc/httpd/ssl


Create certificates for the ca:

>cd /etc/httpd/ssl

>openssl genrsa -des3 -out ca.key 4096

>openssl rsa -in ca.key -out ca.key

>openssl req -new -x509 -days 365 -key ca.key -out ca.crt


Create self-signed certificate for the server (in prod envs sign this with a real CA):

>cd /etc/httpd/ssl

>openssl genrsa -des3 -out server.key 1024

>openssl req -new -key server.key -out server.csr

>cp server.key server.key.org

>openssl rsa -in server.key.org -out server.key

>openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt


Create the client certificate:

>openssl genrsa -des3 -out client.key 1024

>cp client.key client.key.original

>openssl rsa -in client.key -out client.key

>openssl req -new -key client.key -out client.csr


Sign the client certificate with our CA cert:

>openssl x509 -req -days 3650 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt


Create apache docker.conf:

>touch /etc/httpd/conf.d/docker.conf

Add the following to the docker.conf file:

		<VirtualHost *:443>
	        ServerName registry.jufis.net
	        ServerAlias registry.jufis.net
	
	        SSLEngine on
	        SSLCACertificateFile /etc/httpd/ssl/ca.crt
	        SSLCertificateFile /etc/httpd/ssl/server.crt
	        SSLCertificateKeyFile /etc/httpd/ssl/server.key
	
	        Header set Host "registry.jufis.net"
	        RequestHeader set X-Forwarded-Proto "https"
	        ProxyRequests     off
	        ProxyPreserveHost on
	        ProxyPass         / http://127.0.0.1:5000/
	        ProxyPassReverse  / http://127.0.0.1:5000/
	        ErrorLog /etc/httpd/logs/registry-error.log
	        LogLevel warn
	        CustomLog /etc/httpd/logs/registry-access.log combined
	        <Location />
	                Order deny,allow
	                Allow from all
	                #enable this to allow ssl client certificate authentication
	                #SSLRequireSSL
	                #SSLVerifyClient require
	                #SSLVerifyDepth 10
	        </Location>
	        <Location /v1/_ping>
	                Satisfy any
	                Allow from all
	        </Location>
	        <Location /_ping>
	               Satisfy any
	               Allow from all
	        </Location>
		</VirtualHost>

Start apache:

>systemctl start httpd

Start registry:

>systemctl start docker-registry

Check ports are open:

>netstat -an | grep -e 5000 -e 443

Stop firewall:

>systemctl stop firewalld


#Quick guide to install docker on fedora 20 and update to latest

Run the following commands to install docker:

>$ sudo yum -y remove docker

>$ sudo yum -y install docker-io

>$ sudo yum -y update docker-io

>$ sudo systemctl start docker

>$ sudo systemctl enable docker

>$ sudo groupadd docker

>$ sudo chown root:docker /var/run/docker.sock

>$ sudo usermod -a -G docker $USERNAME

Update to latest:

>sudo wget https://get.docker.com/builds/Linux/x86_64/docker-latest -O /usr/bin/docker
