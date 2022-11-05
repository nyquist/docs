# PKI 101

Public Key Infrastructure, aka PKI, is a set of roles, procedures and policies used to manage digital certificates and public key encryption. The end goal is to provide a secure method of exchaning information between parties.

## Public Key Cryptography

### Symmetric Encryption

With symmetric encryption, the same key is used for both encryption and decryption. So if A wants to send B the raw message M, it will use the key K to generate the encrypted message E. When B receives the encrypted message E, it will use the same key K to decrypt it and it will get the raw message M.

<figure><img src=".gitbook/assets/SymmeticKey.png" alt=""><figcaption><p>Symmetric Encryption</p></figcaption></figure>

As you can see, both A and B must have a shared key provided to both of them ahead of time. Hence the term usually used for this aproach: using a pre-shared key (or secret). It's biggest flaw is that secure methods to share the key are hard to find or implement.

### Asymmetric Encryption (PKC)

With asymmetric encryption, aka Public Key Cryptography (PKC), there is one key used in the encryption process and another key used in the decryption process. Therefore, we have a key-pair.

The two keys in the pair are called private-key and public-key. As the name suggests, a public-key can be shared with anyone but the private-key should only be known to the entitiy it belongs to.

<figure><img src=".gitbook/assets/AsymmetricKey.png" alt=""><figcaption><p>Asymmetric Encryption</p></figcaption></figure>

The keys in the key-pair must have certain characteristics in order to be viable. This is done via a mathematical relationship between the private-key and the public-key. The most important characteristic of this relationship is that messages encrypted with the public-key can only be decrypted with the private-key. It doesn't work the other way around and it is computationaly unfeasable to deduce the private-key from the public-key or other data exchanged between the parties. The method used to exchange the key information between parties is known as Diffie-Hellman and RSA (Rivest-Shamir-Adleman) is an implementation of this method.

The DH method is mostly used to secretly agree on a shared key between the parties envolved in a data exchange. Once the shared key is established, it is further used to encrypt the messages exchanged between parties because symmetric encryption is a much simpler process and through this method, it's biggest flaw - sharing the secret - is addressed

#### **Generating a Private Key using openssl**

To create a Private Key using OpenSSL, use this command:

```
openssl genrsa [NUMBITS] [-out OUTPUT_FILE] [-passout PASSOUT] [-des|des3|-aes128|aes192|aes256|...]

NUMBITS = Size of the key. Defaults to 2048.-out 
OUTPUT_FILE = Saves the key to OUTPUT_FILE-passout 
PASSOUT = The key in the OUTPUT_FILE can only be read if the PASSPHRASE is porovided.
If passout is not provided then the user will be asked to enter it. 
Some common methods to provide the PASSPHRASE are:
    pass:PASSPHRASE - provides the passphrase as argument
    file:PASSFILE - provides a file that contains the passphrase
    stdin - asks the user to provide the passphrase
-des|des3|aes128|aes192|aes256|... = Select the method for file encryption.
```

The passphrase is actually used for a symmetric encryption that adds a layer of protection for the private key.

Instead of `openssl genrsa ...` command you should use the newer and more versatile `openssl genpkey -algorithm RSA ...` command. It is meant to suport multiple key types, not just RSA but otherwise follows the same idea while bringin several improvements and more secure defaults.

When looking at an encrypted private key you should see that it includes information about the type of encryption that is used. That information is not present if the key is not protected.

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,1C65746D94033797A4EE53B3FCC8FAF6
4aGNgrMRrgFPlZbHFB9JlPUjhu+sCSUL7VSukEt/slhXpOhpSAKM1b8qdg+25E0F
+kbiijjJcssWiyV+eTm6d/Qemn95DhGhi9H3ffstdziKb6VaKUr8Tfrfg6egsLyK
[...]
```

To read the key from an encrypted file you can use:

```
openssl rsa -in INPUT_FILE [-passin PASSIN] [-out OUTFILE] [-passout PASSOUT]

-passin PASSIN = provide the passphrases needed to read the private key. Follows the same format as PASSOUT
-out OUTFILE = when used it outputs the private key to a file. When it is not used the output will be to screen (stdout)
-passout PASSOUT = privide the passphtrase for the output file.
```

#### **Generating a Public Key using openssl**

The same command can be used to generate a public key based on the private key:

```
openssl rsa -in INPUT_FILE [-passin PASSIN] -pubout [-out OUTFILE]
-passin PASSIN = provide the passphrases needed to read the private key. Follows the same format as PASSOUT
-out OUTFILE = when used it outputs the private key to a file. When it is not used the output will be to screen (stdout)
-pubout = will output the Public Key associated with the input private key
```

#### **Other methods of generating keys**

OpenSSL is not the only tool used to generate keys but it is probably the most versatile.

One of them is `ssh-keygen`, described [here](broken-reference).
