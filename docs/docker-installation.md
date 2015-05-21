#Quick guide on installing docker

I am assuming a linux fedora 20 64bit as a test bed here.

Run the following commands to install docker:

>$ sudo yum -y remove docker

>$ sudo yum -y install docker-io

>$ sudo yum -y update docker-io

>$ sudo systemctl start docker

>$ sudo systemctl enable docker

>$ sudo groupadd docker

>$ sudo chown root:docker /var/run/docker.sock

>$ sudo usermod -a -G docker $USERNAME

Update to latest (optional):

Use this only if want the latest for tests reasons, otherwise stick with the distro files.

>sudo wget https://get.docker.com/builds/Linux/x86_64/docker-latest -O /usr/bin/docker

