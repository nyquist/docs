# PKI 102

## Digital Certificates

One of the most important things when it comes to crytpography is authentication which is the process of verifying that an entity really is who it claims to be. In modern cryptography this is usually done through a digital certificate.

A digital certificate binds an entity's identity with a public key. The binding is performed by a Certificate Authority (CA) through the process of signing.

### Generate a key pair

The process of getting the certificate signed starts with an entity generating a key-pair.

### Create a Certificate Signing Request (CSR)

The next step for the entitiy is to create a Certificate Signing Request (CSR) which will contain the public-key, the identity information, the Distinguished Name (DN) for which it requests the certificate and a section which is generated using the private-key. That is to say the private-key is not part of the CSR but it is used to sign the CSR in order to attest that the request comes from the owner of the public-key.

#### **Creating a CSR using openssl**

```
openssl req -new -key PRIVATE_KEY [-passin PASSIN] -out OUTPUT_FILE
-new = Creates a new CSR
-key PRIVATE_KEY = provides the PRIVATE_KEY needed to generate the Public Key and to sign the request
-passin PASSIN = provides the PASSPHRASE for the PRIVATE_KEY
-out OUTPUT_FILE = saves the request to the provided OUTPUT_FILE
```

If you don't have an existing Private Key you can generate the key and the CSR in the same stept using:

```
openssl req -newkey rsa:NUMBITS -keyout PRIVATE_KEY_FILE -out CSR_FILE [-subj SUBJECT]
-newkey rsa:NUMBITS = will create a RSA key of NUMBITS size.
-keyout PRIVATE_KEY_FILE = will store the generated private key in this file
-out CSR_FILE = will store the genertated CSR in this file
```

Additional identity information will be asked from the user. These make up the DN(Distinguished Name). You can suppress these questsion by providing the reqiured information using the -subj argument.

```
You are about to be asked to enter information that will be incorporated
 into your certificate request.
 What you are about to enter is what is called a Distinguished Name or a DN.
 There are quite a few fields but you can leave some blank
 For some fields there will be a default value,
 If you enter '.', the field will be left blank.
 Country Name (2 letter code) []:US
 State or Province Name (full name) []:NY
 Locality Name (eg, city) []:NYC
 Organization Name (eg, company) []:
 Organizational Unit Name (eg, section) []:
 Common Name (eg, fully qualified host name) []:nyquist.eu
 Email Address []:requester@nyquist.eu
 Please enter the following 'extra' attributes
 to be sent with your certificate request
 A challenge password []:challenge
```

The Common Name (FQDN) is important because this value will have to match your web server address when you you plan to use the certificate for this.

The challenge information requested at the end of the CSR is a password that will be shared between you and the CA.

#### **Creating a CSR using a config file**

For more advanced requests, you can use a config file to provide the input for the CSR. This example will also show how to use SAN(Subject Alternative Names) certificates. A SAN is another name that your identity will be certified for. It's like having multiple different CN. It's not the same as a wildcard certificate.

The config file should follow this format:

```
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req, SAN
prompt = no
[req_distinguished_name]
C = MY_COUNTRY
ST = MY_STATE
L = MY_LOCATION
O = MY_ORGANIZATION
OU = MY_ORG_UNIT
CN = nyquist.eu
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = www.nyquist.eu
DNS.2 = my.nyquist.net
DNS.3 = other.nyquist.net 
IP.1 = X.X.X.X
[SAN]
subjectAltName=DNS:nyquist.eu,DNS:www.nyquist.eu,DNS:my.nyquist.eu,DNS:other.nyquist.eu,IP:X.X.X.X
```

Of course, you should replace the values provided for desitnguished name, common name and alt name. After that, generate the CSR with:

```
openssl req -new -key PRIVATE_KEY [-passin PASSIN] -out OUTPUT_FILE -config CFG_FILE -extensions v3_req -extensions SAN
```

#### **Reading the contents of a CSR**

To read the contents of a CSR you can use:

```
openssl req -in INPUT_CSR -text
Certificate Request:
     Data:
         Version: 0 (0x0)
         Subject: C=US, ST=NY, L=NYC, CN=nyquist.eu/emailAddress=requester@nyquist.eu
         Subject Public Key Info:
             Public Key Algorithm: rsaEncryption
                 Public-Key: (2048 bit)
                 Modulus:
                     00:d4:69:15:3e:2f:a3:c6:4a:10:ba:b4:b6:cc:65:
                     [...]
                     97:8f
                 Exponent: 65537 (0x10001)
         Attributes:
             challengePassword        :unable to print attribute
     Signature Algorithm: sha256WithRSAEncryption
          39:5f:d5:0b:52:5f:16:e9:61:f7:d8:ba:65:42:80:41:de:04:
          [...]
          ba:3b:bc:4c
 -----BEGIN CERTIFICATE REQUEST-----
 MIICwTCCAakCAQAwYjELMAkGA1UEBhMCVVMxCzAJBgNVBAgMAk5ZMQwwCgYDVQQH
 [...]
 XRKFIKClhvbgondwsCojca6f3dp2I6UQj1aim+RoH++yuju8TA==
 -----END CERTIFICATE REQUEST-----
```

### Signing the CSR and generating the certificate

#### **CA signed Certificates**

The CSR could go through a Registration Authority (RA) that verifies the identity of the entitiy against the provided CSR before reaching the Certificate Authority (CA) for signing. Most of the times, RA and CA are the same so it all looks like a single step.

To sign the request, a CA owner should use this command:

```
openssl ca -keyfile CA_PRIVATE_KEY -cert CA_CERTIFICATE -in CSR_FILE -out CERTIFICATE_FILE [-extensions v3_req -extensions SAN  -extfile CONFIG_FILE]
-keyfile CA_PRIVATE_KEY: points to the CA private key
-cert CA_CERTIFICATE: points to the CA root certificate
-in CSR_FILE: points to the CSR for the new certificate we want signed
-out CERTIFICATE_FILE: where to store the output certificate
-extensions EXT: optionally, if you want to include certificate extensions
-extfile CONFIG_FILE: optionally if you want to read extensions from a config file
```

#### **Self Signed Certificates**

Instead of going through a CA, you can also sign your own requests and generate a "self-signed certificate". That's the easy way, but also less trusted. By default self-signed certificates are not trusted by most modern browsers.

To sign a CSR (and get a self-signed certificate) you can use the following openssl command:

```
openssl req -x509 -in CSR_FILE -key CA_RSA  -days DAYS -out OUT_CRT
-x509 = Request an x509 certificate
-in CSR_FILE = input of the CSR request
-key CA_RSA = private key of the signing entity.
-days DAYS = how many days should the certificate be available
-out OUT_CRT = certificate output file
```

After it is signed, the certificate is exported to the requester. The standard for these certificates is registered as [X.509](https://en.wikipedia.org/wiki/X.509) but there are different formats for the certificate (.pem, .p12, .cer, .crt and others). The p12 format allows adding an additional passphrase to the certificate so it is not readable without knowing the passphrase.

To decode the information of a crt file you can use:

```
openssl x509 -text -in CRT_FILE
 Certificate:
     Data:
         Version: 1 (0x0)
         Serial Number: 18430850791973880580 (0xffc78924fbd48304)
     Signature Algorithm: sha256WithRSAEncryption
         Issuer: C=US, ST=NY, L=NYC, CN=nyquist.eu/emailAddress=requester@nyquist.eu
         Validity
             Not Before: Nov 19 20:05:57 2019 GMT
             Not After : Nov 18 20:05:57 2020 GMT
         Subject: C=US, ST=NY, L=NYC, CN=nyquist.eu/emailAddress=requester@nyquist.eu
         Subject Public Key Info:
             Public Key Algorithm: rsaEncryption
                 Public-Key: (2048 bit)
                 Modulus:
                     00:d4:69:15:3e:2f:a3:c6:4a:10:ba:b4:b6:cc:65:
                     [...]
                     97:8f
                 Exponent: 65537 (0x10001)
     Signature Algorithm: sha256WithRSAEncryption
          8d:78:f7:7d:03:c7:d3:4c:cb:6a:54:f6:e9:e7:dc:60:9f:61:
          [...]
          e1:aa:10:9b
 -----BEGIN CERTIFICATE-----
 MIIDQDCCAigCCQD/x4kk+9SDBDANBgkqhkiG9w0BAQsFADBiMQswCQYDVQQGEwJV
 [...]
 1jeF4YmLklcgo7TIJCROwuGqEJs=
 -----END CERTIFICATE-----
```

### Chain of Trust

A certificate chain is a list of certificates that have the following characteristics:

* The issuer of each certificate matches the subject of the next certificate in the list.
* The private-key used to sign each certificate can be verified using the public-key of the next certificate in the list
* The last certificate in the list is trusted through other methods (manually set to trust, embedded in the system, etc)

By using this chain of trust, the certificate provided by an entitiy is verified by another entity and the chain goes from the certificate under verification, through intermediate certificates, up to the CA's root certificate.

In some simple scenarios you can provide your own certificate by performing the signing process yourself. In this case the certificate is called "self-signed" but others may not have a high level of trust in this kind of certificate.

Web browsers usually include intermediate certificates provided by well-known CAs and set to trust in order to enable the verification process to take advantage of the chain up to the well-known CAs without passing their root certificates to the public.

A useful command to check the certificate of a website is:

```
openssl s_client -showcerts -connect google.com:443
```
