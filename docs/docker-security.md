#Setup docker daemon for secure remote access

If you have not installed docker follow this **[link here](docker-installation.md)**

##Docker security

In our initial installation we used docker daemon to listen to an HTTP socket in all interfaces. In this section we will secure the deamon to listen to a secure socket and only allow clients that their certificate is signed only by the CA that the docker deamon trusts.

##CA Certificate setup

*NOTE: I assume that my docker daemon host has a DNS record named docker.jufis.net*
*NOTE: I assume that my docker client host has a DNS record named client.jufis.net*

Let's create the private/public keys with the **user** that we will also run the docker client for the sake of the example:

(this can also be the docker daemon *server" machine as well)

>cd ~

>mkdir -pv ~/.docker

>cd ~/.docker

>openssl genrsa -aes256 -out ca-key.pem 2048

Output:
<pre>
Enter pass phrase for ca-key.pem: **stuff**
Verifying - Enter pass phrase for ca-key.pem: **stuff**
</pre>

>openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

Output:
<pre>
Enter pass phrase for ca-key.pem: **stuff**
...
Country Name (2 letter code) [XX]:GR
State or Province Name (full name) []:Attika
Locality Name (eg, city) [Default City]:Melissia
Organization Name (eg, company) [Default Company Ltd]:Jufis.net
Organizational Unit Name (eg, section) []:Dev Ops
Common Name (eg, your name or your server's hostname) []:docker.jufis.net
Email Address []:jufis@jufis.net
</pre>

Now that we have created the CA private/public keys, we will create the server key and a certificate request that we will sign it later on with the ca we just created. 

*Make sure that "Common Name" (i.e., server FQDN or YOUR name) matches the hostname you will use to connect to Docker. If you have a DNS record registered for the docker host you MUST use this name here.*

>openssl genrsa -out server-key.pem 2048

Output:
<pre>
Generating RSA private key, 2048 bit long modulus
..........................+++
...........+++
e is 65537 (0x10001)
</pre>

>openssl req -subj "/CN=docker.jufis.net" -new -key server-key.pem -out server.csr

To allow connections from hosts using ip addresses (eg:127.0.0.1,192.168.1.7) as well as DNS names we can create an extension file as follows:

>echo subjectAltName = IP:192.168.1.7,IP:127.0.0.1 > extfile.cnf

>openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf

Output:
<pre>
Signature ok
subject=/CN=docker.jufis.net
Getting CA Private Key
Enter pass phrase for ca-key.pem: **stuff**
</pre>

For all docker clients we will use the following procedure to create their client key/certificate:

>openssl genrsa -out key.pem 2048

Output:
<pre>
Generating RSA private key, 2048 bit long modulus
....+++
.....+++
e is 65537 (0x10001)
</pre>

Create certificate request to allow CA to sign our client certificate:

>openssl req -subj '/CN=client.jufis.net' -new -key key.pem -out client.csr

To make the key suitable for client authentication, create an extensions config file:

>echo extendedKeyUsage = clientAuth > extfile.cnf

Now let's sign the client key with the CA certificate:

>openssl x509 -req -days 365 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf

Output:
<pre>
Signature ok
subject=/CN=client.jufis.net
Getting CA Private Key
Enter pass phrase for ca-key.pem: **stuff**
</pre>

Remove certificate requests:

>rm -rf *.csr

Protect your keys:

>chmod -v 0400 ca-key.pem key.pem server-key.pem

Output:
<pre>
mode of ‘ca-key.pem’ changed from 0664 (rw-rw-r--) to 0400 (r--------)
mode of ‘key.pem’ changed from 0664 (rw-rw-r--) to 0400 (r--------)
mode of ‘server-key.pem’ changed from 0664 (rw-rw-r--) to 0400 (r--------)
</pre>

>chmod -v 0444 ca.pem server-cert.pem cert.pem

Output:
<pre>
mode of ‘ca.pem’ changed from 0664 (rw-rw-r--) to 0444 (r--r--r--)
mode of ‘server-cert.pem’ changed from 0664 (rw-rw-r--) to 0444 (r--r--r--)
mode of ‘cert.pem’ changed from 0664 (rw-rw-r--) to 0444 (r--r--r--)
</pre>

##Docker daemon setup for security

Stop docker deamon first:

>sudo systemctl stop docker.service

Edit docker daemon configuration:

>sudo vi /etc/sysconfig/docker

Comment out your existing OPTIONS variable and replace with the following:

<pre>
OPTIONS='--tlsverify --tlscacert=/root/.docker/ca.pem --tlscert=/root/.docker/server-cert.pem --tlskey=/root/.docker/server-key.pem -H=0.0.0.0:2376'
</pre>

Copy all certs/keys from local user (eg. jufis in my example) to root user that the daemon uses to run:

>cp -rf /home/jufis/.docker /root/

Start docker deamon:

>sudo systemctl start docker.service

Check that the daemon is up and running:

>ps ax |grep docker

<pre>
 7510 ?        Ssl    0:00 /usr/bin/docker -d --tlsverify --tlscacert=/root/.docker/ca.pem --tlscert=/root/.docker/server-cert.pem --tlskey=/root/.docker/server-key.pem -H=0.0.0.0:2376 --insecure-registry 0.0.0.0:5000
</pre>

If this is not running check your paths and also check syslog in order to identify the reason of failure:

>tail /var/log/messages

Check that the port listening is 2376 and *not* 2375:

>netstat -an |grep -i listen|grep 2376

Output:
<pre>
tcp6       0      0 :::2376                 :::*                    LISTEN
</pre>


##Docker client machine setup for default security

For every client that is required to connect to remote docker deamon securely you need to perform the following:

>login with the user that will run docker

>cd ~

>mkdir -pv ~/.docker

>Copy the above {ca,cert,key}.pem files to ~/.docker

Edit /etc/bashrc and append at the end:

>sudo vi /etc/bashrc

>export DOCKER_HOST=tcp://docker.jufis.net:2376 DOCKER_TLS_VERIFY=1

Close and restart your terminal session with the same user or re-source bashrc settings.

Now test the secure docker connection:

>docker version

Output:
<pre>
[jufis@jufis ~]$ docker version
Client version: 1.6.0
Client API version: 1.18
Go version (client): go1.4.2
Git commit (client): 4749651
OS/Arch (client): linux/amd64
Server version: 1.6.0
Server API version: 1.18
Go version (server): go1.4.2
Git commit (server): 4749651
OS/Arch (server): linux/amd64
</pre>

Test it around especially if you have some images running around:

<pre>
[jufis@jufis ~]$ docker images
REPOSITORY                                  TAG                         IMAGE ID            CREATED             VIRTUAL SIZE
registry.jufis.net:443/docker-rest-client   TAG_V1_04_MAY_2015_master   c00ba3434e03        6 days ago          596.5 MB
[jufis@jufis ~]$ echo $DOCKER_
$DOCKER_HOST        $DOCKER_TLS_VERIFY  
[jufis@jufis ~]$ echo $DOCKER_TLS_VERIFY 
1
[jufis@jufis ~]$ echo $DOCKER_HOST
tcp://docker.jufis.net:2376
</pre>

##References

http://docs.docker.com
http://docs.docker.com/articles/https/
