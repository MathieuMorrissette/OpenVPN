## Requirements
 - openvpn 
 - easy-rsa
## Generate a Certificate Authority
We need a certificate authority to create certificates for client. 
In my use case the certificate authority server will be the same as the
OpenVPN server.

We start by first creating the directory that will contain our certificates.
With the following command :

```make-cadir /etc/openvpn/easy-rsa/```

```cd /etc/openvpn/easy-rsa```

Then we must setup the variables for our certificate authority.

Edit those vars to your likings :

```nano /etc/openvpn/easy-rsa/vars```

```
export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"
```

They will be used by easy-rsa when generating the certificates
You must do :

``` source vars```

If you get some openssl error, you can edit this variable to point directly
to the OpenSSL config file.

```export KEY_CONFIG=$EASY_RSA/openssl-1.0.0.cnf```

To make sure we're running in a clean environment 

```./clean-all```

Now we can build our root certificate (this can take a long time depending on the key length)

```./build-ca```

## Generating Server Certificate

Now we must generate the server certificate, with the following command
(replace server with a name of your likings)

```./build-key-server server```

Follow the on-screen instructions

## Generating a strong diffie-hellman key for encryption

```./build-dh```

Afterwards, we can generate an HMAC signature to strengthen the server's TLS integrity verification capabilities:

```openvpn --genkey --secret keys/ta.key```

## Generating clients keys

Make sure to still be in the folder with your variables still configured.

```./clean-all```

```./build-key client1```

## Configuring the OpenVPN Server




### Source
[https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04)
