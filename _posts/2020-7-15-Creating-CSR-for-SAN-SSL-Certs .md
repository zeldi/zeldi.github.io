---
layout: post
title: Creating CSR and Key File for a SAN Certificate with Subject Alternate Names using OpenSSL
permalink: creating-san-csr
date: 2023-09-15
tags: ["csr","san","ssl"]
---


Creating a Certificate Signing Request (CSR) and Key File for a Subject Alternative Name (SAN) certificate with OpenSSL involves a few steps. SAN certificates allow you to secure multiple domains or subdomains with a single certificate. Here's how you can generate a CSR and Key File with OpenSSL for a SAN certificate:

**1. Install OpenSSL**: If you don't have OpenSSL installed, you'll need to do that first. You can download it from the OpenSSL website or use a package manager specific to your operating system.

**2. Generate a Private Key**: Run the following command to generate a private key `server.key`. You may change the filename with desired name for your private key

```bash
openssl genrsa -out server.key 2048
```

**3. Create a Configuration File**: Create a configuration file (e.g., `san_config.cnf`) that defines the subject alternative names. Here's an example of what the contents of the `san_config.cnf` file might look like:
```
[req]
default_bits = 2048  
encrypt_key = no  
default_md = sha256  
utf8 = yes  
string_mask = utf8only  
prompt = no  
distinguished_name = req_distinguished_name  
req_extensions = req_ext

[req_distinguished_name]
countryName = MY
stateOrProvinceName = Selangor
localityName = CyberJaya
organizationName = <organization-name>
organizationalUnitName = <org-unit-name>
commonName = app1.mysite.com

[req_ext]  
subjectAltName = @alt_names

[alt_names]
DNS.1=app1.mysite.com
DNS.2=alt-app1.mysite.com
DNS.3=app2.yoursite.com
```

> __NOTE__
> * Modify the `DNS.1`, `DNS.2`, etc., lines to list the domain names and subdomains you want to include in your SAN certificate.
> * `commonName` Should match a SAN under alt_names

**4. Generate the CSR**: Run the following command to generate the CSR using the private key and configuration file:
```bash
openssl req -new -sha256 -out server.csr -key server.key -config san_config.cnf
```
Replace `server.key` with the name of your private key file and `server.csr` with the desired CSR filename.

**5. Review and Verify**: Review the CSR file (`server.csr`) to ensure that it contains the correct SANs:
```bash
openssl req -text -noout -in server.csr

Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: C = MY, ST = Selangor, L = CyberJaya, O = myOrgzanization, OU = organziation-name, CN = app1.mysite.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:e6:2d:7e:f0:46:3a:4d:7f:07:37:98:7f:c1:36:
                    d5:e2:12:46:d4:17:c9:0d:1d:68:81:c6:b6:5d:41:
                    89:1c:5d:07:4f:29:34:b5:19:54:30:83:dd:fc:2f:
                    c2:7a:b0:ec:6f:f2:54:a8:c4:b6:5d:38:3a:15:32:
                    90:94:62:1d:28:b7:0c:d2:e8:c6:91:62:f9:32:67:
                    22:48:28:70:11:77:e3:d2:16:ae:1a:74:e4:a7:95:
                    49:50:6a:0f:9e:a5:1b:2d:c0:c5:5a:48:37:61:0f:
                    32:ec:23:7f:a8:92:b2:fc:df:90:99:a2:65:0e:87:
                    7f:17:3e:ac:03:0d:d2:06:9c:9c:34:b4:e0:44:72:
                    04:25:27:21:d6:6c:37:78:dc:a8:03:c3:ef:f0:74:
                    29:e8:ff:d5:d8:be:11:52:0c:d7:3a:4e:79:4e:60:
                    b4:93:f5:6a:4e:c5:94:30:d9:bb:9d:06:dc:ff:24:
                    98:11:0e:89:cb:c4:70:e6:9d:16:e9:e3:e4:ab:5a:
                    2b:08:49:83:ab:fa:f5:57:43:5a:dd:e4:56:f9:58:
                    0e:86:26:d8:16:94:72:78:1f:6e:d9:e5:9c:96:bb:
                    cd:04:32:92:d4:74:8c:5a:13:18:4d:26:c5:0c:d7:
                    1a:72:8c:19:0e:3f:b0:87:72:95:06:0c:fa:e7:08:
                    b3:05
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name: DNS:app1.mysite.com, DNS:alt-app1.mysite.com, DNS:app2.yoursite.com
```

**6. Submit the CSR**: Submit the CSR to a certificate authority (CA) to obtain your SAN certificate. The CA will provide you with the SAN certificate file once they have verified your domain ownership.


> __NOTE__
> * Remember to keep your private key (`server.key`) secure and never share it with anyone. The CSR (`server.csr`) is safe to share with the CA for certificate issuance. 
> * Once you receive your SAN certificate, you can install it along with the private key on your web server or application.


