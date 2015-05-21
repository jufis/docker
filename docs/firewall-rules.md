#Firewall Rules

To secure docker registry:

>setup your firewall to allow connections only from docker daemon ip/port to your.host:5000

To further secure docker daemon on top of TLS:

>setup your firewall to allow connections only from docker clients ip/port to your.host:2376
