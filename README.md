POC: LDAP-based key authentication
==================================

To reproduce the test VM, run
```
vagrant up
```

## Synopsis

The POC example demonstrates how ssh login can be controlled via keys
and POSIX groups, maintained in an LDAP directory. In this example, we
consider 3 users:

- user `good`, administratively allowed to login with ssh keypair
- user `bad`, for whom the keypair exists but who is administratively
barred from logging in
- user `ugly`, who is allowed to log in but for whom a private key is
suspected to have been compromized and thus has been revoked.

Note that the example does not demonstrate best practices, merely
illustrates that the security administrator has the capability to
control the scope and means of access. The example does not obviate
the need to cull inactive users from allowed lists, nor to purge
leaked ssh keys.

### Example

user `good` is allowed to log in:

```
$ ssh localhost -p 2222 -l good -i id_good.rsa
```

### Example

user `bad` is denied access:
```
$ ssh localhost -p 2222 -l bad -i id_bad.rsa
```

#### sshd logs
```
Sep  9 06:27:21 vagrant sshd[4865]: User bad from 10.0.2.2 not allowed because a group is listed in DenyGroups
```

### Example

user `ugly` is allowed to log in with a clean key but not with a
compromized one:

```
$ ssh localhost -p 2222 -l ugly -i id_ugly_leaked.rsa
```

#### sshd logs
```
Sep  9 06:48:53 vagrant sshd[3278]: Failed publickey for ugly from 10.0.2.2 port 49175 ssh2: RSA 7e:26:63:01:37:1e:8d:27:67:ac:80:06:61:7c:81:a5
Sep  9 06:48:53 vagrant sshd[3278]: error: WARNING: authentication attempt with a revoked RSA key 5b:a4:df:16:ed:cf:5e:b9:dc:a2:f1:e5:43:77:25:fa
Sep  9 06:48:53 vagrant sshd[3278]: Failed publickey for ugly from 10.0.2.2 port 49175 ssh2: RSA 5b:a4:df:16:ed:cf:5e:b9:dc:a2:f1:e5:43:77:25:fa
```

```
$ ssh localhost -p 2222 -l ugly -i id_ugly.rsa
```

#### sshd logs
```
Sep  9 06:24:29 vagrant sshd[4709]: Starting session: shell on pts/6 for ugly from 10.0.2.2 port 65297
```
