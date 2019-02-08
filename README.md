## Requirements
 - openvpn 
 - easy-rsa
## Generate a Certificate Authority
We need a certificate authority to create certificates for client. 
In my use case the certificate authority server will be the same as the
OpenVPN server.

We start by first creating the directory that will contain our certificates.
With the following command :

```sudo make-cadir /etc/openvpn/easy-rsa/```

Then we must setup the variables for our certificate authority.

Edit those vars to your likings :

```sudo nano /etc/openvpn/easy-rsa/vars```

```
export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"
```

They will be used by easy-rsa when generating the certificates

### Source
[https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04)
