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

If you don't want a password just leave it blank when prompted

## Configuring the OpenVPN Server

We first need to copy the keys we generated to the openvpn directory 

```cd keys```
```cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn```

Then we need to get the sample configuration file to use as a base

```gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf```

Edit the configuration file

```nano /etc/openvpn/server.conf```

First, find the HMAC section by looking for the tls-auth directive. Remove the ";" to uncomment the tls-auth line. Below this, add the key-direction parameter set to "0":

```
tls-auth ta.key 0 # This file is secret
key-direction 0
```

Next, find the section on cryptographic ciphers by looking for the commented out cipher lines. The AES-128-CBC cipher offers a good level of encryption and is well supported. Remove the ";" to uncomment the cipher AES-128-CBC line:

```cipher AES-128-CBC```

Below this, add an auth line to select the HMAC message digest algorithm. For this, SHA256 is a good choice:

```auth SHA256```

Finally, find the user and group settings and remove the ";" at the beginning of to uncomment those lines:

```
user nobody
group nogroup
```

### Optional

We can push gateway and DNS servers to the VPN clients

Find the redirect-gateway section and remove the semicolon ";" from the beginning of the redirect-gateway line to uncomment it:

```push "redirect-gateway def1 bypass-dhcp"```

Just below this, find the dhcp-option section. Again, remove the ";" from in front of both of the lines to uncomment them:

```
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

I recommend using port 443 and protocol tcp as they go through firewalls the easiest

## Allow Forwarding of traffic

First, we need to allow the server to forward traffic. This is fairly essential to the functionality we want our VPN server to provide.

We can adjust this setting by modifying the /etc/sysctl.conf file:

```nano /etc/sysctl.conf```

Inside, look for the line that sets net.ipv4.ip_forward. Remove the "#" character from the beginning of the line to uncomment that setting:

```net.ipv4.ip_forward=1```

To read the file and adjust the values for the current session, type:

```sysctl -p```

## Allow traffic to the internet

We must add a rule in iptables that will forward traffic from nat to eth0

```iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE```

iptables don't save by themselves so we can install this 
- iptables-persistent

```iptables-save > /etc/iptables/rules.v4```
```iptables-restore < /etc/iptables/rules.v4```

## Client configuration



### Source
[https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04)
