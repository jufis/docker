#Quick guide on installing docker

I am assuming a linux fedora 20 64bit as a test bed here. This setup doesn't provide any guarantee securities use [docker security link](docker-security.md) to improove security dramatically.

Run the following commands to install docker:

>$ sudo yum -y remove docker

>$ sudo yum -y install docker-io

>$ sudo yum -y update docker-io

>$ sudo systemctl start docker

>$ sudo systemctl enable docker

>$ sudo groupadd docker

>$ sudo chown root:docker /var/run/docker.sock

>$ sudo usermod -a -G docker $USERNAME

##Disable SELinux

Edit selinux config:

>vi /etc/sysconfig/selinux

Disable it:

>SELINUX=disabled

Reboot linux.

##Configure daemon to listen to all interfaces

Edit docker configuration and ensure that the following are applied:

>vi /etc/sysconfig/docker.conf 

>OPTIONS='-H tcp://0.0.0.0:2375'

>DOCKER_CERT_PATH=/etc/docker

>INSECURE_REGISTRY='--insecure-registry 0.0.0.0:5000'

>setsebool -P docker_transition_unconfined

>GOTRACEBACK=crash

Update to latest (optional):

Use this only if want the latest for tests reasons, otherwise stick with the distro files.

>sudo wget https://get.docker.com/builds/Linux/x86_64/docker-latest -O /usr/bin/docker

