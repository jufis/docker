#Setup docker daemon for secure remote access

##Daemon setup

##Certificate setup

>mkdir /etc/docker/certs.d/registry.jufis.net:443

>cd /etc/docker/certs.d/registry.jufis.net:443

>cp /etc/httpd/ssl/ca.crt .

>cp /etc/httpd/ssl/client.crt client.cert

>cp /etc/httpd/ssl/client.key .