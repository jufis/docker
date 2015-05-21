#Setup docker daemon for secure remote access

If you have not installed docker follow this **[link here](docs/docke-installation.md)**

##Daemon setup

##Certificate setup

>mkdir /etc/docker/certs.d/registry.jufis.net:443

>cd /etc/docker/certs.d/registry.jufis.net:443

>cp /etc/httpd/ssl/ca.crt .

>cp /etc/httpd/ssl/client.crt client.cert

>cp /etc/httpd/ssl/client.key .

##References

http://docs.docker.com/articles/https/
