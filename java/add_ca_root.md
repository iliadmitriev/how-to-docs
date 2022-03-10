Root CA certificates is stored in file

`ls $(brew --prefix)/opt/openjdk@11/libexec/openjdk.jdk/Contents/Home/lib/security/cacerts`

default password is `changeit`


Find keytool util

```shell
which keytool
```

List CA root certificates

```shell
keytool -list -cacerts
```

Add certificate to java root using keytool (use password `changeit`)

```shell
keytool  -importcert -trustcacerts -cacerts -file self_signed_ca.pem -alias 'self signed CA root'

```
