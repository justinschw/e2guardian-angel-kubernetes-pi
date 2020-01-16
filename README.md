e2guardian-angel-kubernetes-pi
======================
This repository contains the yaml definitions for a kubernetes cluster
containing a squid and e2guardian service that run on a raspberry pi.

The docker images  used are as follows:
jusschwa/e2guardian-rpi
jusschwa/docker-squid-sslbump-rpi

The cluster requires use of a CIFS Flexvolume plugin. Details here:

https://github.com/fstab/cifs

This also requires setting up a samba share and creating secrets for it,
per the instructions on the github page for that driver.

Currently for the service I am using NodePort.

Usage
======================
```sh
kubectl create -f e2guardian-angel.yml
kubectl apply -f e2guardian-service.yml
```

Usage (Proxy)
======================
Import your CA certificate (stored on your samba share) into your browser.
Point your proxy IP/port to (node IP):31280 where node IP is the public
IP of any node in your cluster

Note
======================
This does not include the default configuration required to make e2guardian
and squid talk to each other. That will be provided in a separate
repository. Details will be included later.

There are two sets of secrets that are needed for this cluster:
1. Certificate and key for squid MITM decryption (secretName: squid-cert-key)
2. CIFS secret (secretName: cifs-secret) created during the driver installation
You have to create these manually.

The samba shares referred to in the yaml are as follows:
//kubernetes-samba-share/e2guardian-etc
//kubernetes-samba-share/squid-etc

When you set up your samba shares, you will have to configure these two shares
to authenticate using the secrets mentioned before. These shares should contain
the files that would go under /etc/e2guardian and /usr/local/squid/etc,
respectively.

In your smb.conf you should have something like the following:
```
[global]

...
lanman auth = yes
ntlm auth = yes
client ntlmv2 auth = yes

...

[e2guardian-etc]
path = /path/to/squid-e2guardian-config/e2guardian
valid users = @sambashare
browsable = yes
writable = no
read only = yes

[squid-etc]
path = /path/to/squid-e2guardian-config/squid/etc
valid users = @sambashare
browsable = yes
writable = no
read only = yes
```

Create e2guardian user and set the samba password like so:
```sh
sudo smbpasswd e2guardian
```

Once your samba shares are set up properly, you should add
a DNS entry for kubernetes-samba-share and point it to the
IP where samba is running. You can also just add it to
the /etc/hosts of each of your nodes.