# SSH Client 101

## On the client host

### Connect to a (remote) host

To connect to a SSH server use

```
ssh [-i IDENTITY_FILE] [-l USER |USER@]HOST
-l USER or USER@: specify the username to use. If missing, the current username will be used.
-i IDENITY_FILE: use the identity (private key) described in the file.
```

If no identity file is specified, the client will try to look for one in

```
~/.ssh/id_rsa
```

. That is for RSA keys. For other key types there are corresponding default locations:

```
id_dsa, id_ecdsa, id_ed25519
```

. If no identity is found the user might be asked to provide a password.

To automate the selection of identities, the

```
~/.ssh/config
```

file can be used to provide an idenity for each host:

```
Host HOST1
     IdentityFile IDENTITY_FILE1
Host HOST2
     IdentityFile IDENTITY_FILE2
```

### Managing SSH Keys

#### **ssh-keygen**

There are a few methods to create keys, depending on OS. Some OpenSSL examples are available [here](broken-reference). But you can also you ssh-keygen which comes bundled with the ssh client. To generate keys with ssh-keyge, use:

```
ssh-keygen [-b NUMBITS] [-t dsa|ecdsa|ed25519|rsa] [-N PASSPHRASE] [-C COMMENT] [-f OUTPUT_KEYFILE]
-b NUMBITS: The key size in bits. Default 2048
-t dsa|ecdsa|ed25519|rsa: Chose the key type. Default is RSA.
-N PASSPHRASE: set addtional passphrase to protect the private key
-C COMMENT: set optional comment to provide additional information about the key
-f OUTPUT_KEYFILE: If not specified, the default location will be used: ~/.ssh/id_rsa (or id_dsa, id_ecdsa, id_ed25519)
```

Both a private key (id\_rsa) and the corresponding public key (id\_rsa.pub) will be created.

Best practice is to generate one key pair on each host you use to connect and to add the public key to the hosts you want to connect to. You shouldn't copy the private key on different hosts.

#### **ssh-agent and ssh-add**

ssh-agent is a program that holds private keys. It usually starts at the beginning of a user session and starts with no keys.

You can add keys to the ssh-agent using ssh-add

```
ssh-add [IDENTITY_FILE] [-t LIFE] ...
IDENTITY_FILE: file that holds the private key. If no file is provided, the default locations (~/.ssh/id_rsa, ...) will be added to the agent.
-t LIFE: set the maximum lifetime of the key in the agent. After lifetime expires, the key is removed from the agent.
```

You can check the list of existing keys with

```
ssh-add -l
```

One additional feature of the agent is that it can forward keys when you ssh on to other hosts. This is useful in that you don't have to copy your private keys anywhere else or you don't have to generate new sets of keys on other hosts. Using the SSH connection the agent can provide key information to the agent running on the remote host.

To do this you either specify this at the moment of connection with

```
ssh -A USER@HOST
```

add it in the ssh client config file:

```
Host HOST1
     ForwardAgent yes
    
```

### On the server host

Presuming the server allows Public Key Authentication, the user can add its own public keys in

```
~/.ssh/authorized_keys
```

. Once there, key based authentication can take place.

You can also use ssh-copy-id to securely copy the public key from one host to another.
