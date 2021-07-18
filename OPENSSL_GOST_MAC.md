# Openssl with GOST 2012 for macOS (including M1 Apple Silicone)

How-to documentation on GOST R 34.10-2012 digital signature algorithm
for macOS (including M1 Apple Silicone)

Using:
* [brew](https://brew.sh)
* [OpenSSL 1.1](https://www.openssl.org)
* [Gost engine library](https://github.com/gost-engine/engine)

# Install

## Prepare

Install brew https://brew.sh

Install macOS build tools
```shell
xcode-select --install
```

install brew packages
```shell
brew install cmake pkg-config openssl@1.1
```

configure pkg-config tool variable to link gost-engine with brew version of openssl
```shell
export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:$(brew --prefix)/opt/openssl@1.1/lib/pkgconfig
# check flags
pkg-config --cflags openssl
```

clone gost-engine from github repository for openssl version 1.1.1
```shell
git clone git@github.com:gost-engine/engine.git
cd engine
git checkout openssl_1_1_1
```

## Build

Create build directory and build gost-engine
```shell
mkdir build; cd build
cmake ../
make
cd bin
ls -al ./
```

## Install

Save openssl engines directory path to ENGINESDIR variable
```shell
ENGINESDIR=$($(brew --prefix)/opt/openssl@1.1/bin/openssl version -e |
                sed  's/.*\"\(.*\)\".*/\1/')
ls -la ${ENGINESDIR}
```

Copy library file to openssl engines directory
```shell
cp gost.1.1.dylib ${ENGINESDIR}/gost.dylib
ls -la ${ENGINESDIR}
```

## Configure

[Sample configuration](https://github.com/gost-engine/engine/blob/master/example.conf)

Save `openssl.cnf` config path to OPENSSLCFG variable

```shell
OPENSSLCFG=$($(brew --prefix)/opt/openssl@1.1/bin/openssl version -d |
                sed  's/.*\"\(.*\)\".*/\1/')/openssl.cnf
ls -la ${OPENSSLCFG}
```

Add first line `openssl_conf = openssl_def` to `openssl.cnf` config

```shell
grep -q '^openssl_conf' $OPENSSLCFG ||
sed -i '' '1i\'$'\n''openssl_conf = openssl_def'$'\n' $OPENSSLCFG
```

Add to the end of `openssl.cnf` config

```shell
cat >> $OPENSSLCFG << _EOF_
[openssl_def]
engines = engine_section

[engine_section]
gost = gost_section

[gost_section]
engine_id = gost
default_algorithms = ALL
CRYPT_PARAMS = id-Gost28147-89-CryptoPro-A-ParamSet
_EOF_
```

### Create alias

create shortcut alias for brew version of openssl `openssl@1.1`
```shell
alias openssl@1.1=$(brew --prefix)/opt/openssl@1.1/bin/openssl
```

## Check installation

```shell
openssl@1.1 engine
openssl@1.1 engine gost -c -t
```

Both commands should output:
```
(gost) Reference implementation of GOST engine
```

# Usage

List cipher algorithm
```shell
openssl@1.1 list --cipher-algorithms
# or
openssl@1.1 enc -engine gost -ciphers
```

List digest algorithms
```shell
openssl@1.1 list --digest-algorithms
# or
openssl@1.1 dgst -list 
```

## Generate key and certificate

Generate gost 2012 private key 
```shell
openssl@1.1 genpkey -algorithm gost2012_256 \
            -pkeyopt paramset:TCB -out ca.key
```
Generate gost 2012 certificate
```shell
openssl@1.1 req -new -x509 -md_gost12_256 -days 365 \
            -key ca.key -out ca.cer
            
# or

openssl@1.1 req -new -x509 -md_gost12_256 -days 365 \
   -subj "/C=RU/ST=Russia/L=Moscow/O=Internet/OU=Internet CA/CN=Internet CA Root" \
   -key ca.key -out ca.cer
```

[Read for more](https://github.com/gost-engine/engine/blob/master/README.gost)

## Check certificate

```shell
openssl@1.1 x509 -in ca.cer -text -noout
```

## Check if a private key matches a certificate

Generate public key md5hash for private key
```shell
openssl@1.1 pkey -in ca.key -pubout -outform pem | openssl@1.1 md5
```

Generate certificate public key md5hash
```shell
openssl@1.1 x509 -in ca.cer -noout -pubkey | openssl@1.1 md5
```

Both md5 hashes should match

## Sign and check signature

Create test file `input.txt`

```shell
cat > input.txt << _TXT_
Sed dictum sapien in scelerisque accumsan. Ut ac vehicula tortor, luctus rutrum augue.
Ut rhoncus blandit nisi, a tincidunt orci pulvinar non. Quisque at rutrum est.
Ut a aliquam felis. Nam et vestibulum tortor. Aliquam ut rhoncus nulla. Nulla tincidunt,
diam eu eleifend eleifend, est orci lobortis sem, eu placerat nibh ligula at risus.
_TXT_
```

Sign without detaching content
```shell
openssl@1.1 smime -sign -nodetach -engine gost  \
            -binary -md md_gost12_256 -in input.txt \
            -signer ca.cer -inkey ca.key -out output.out -outform DER
```

Verify signature and output original content
```shell
openssl@1.1 smime -verify -noverify -engine gost  \
            -binary -md md_gost12_256 -in output.out -inform DER \
            -signer ca.cer -inkey ca.key -out output.txt
```

## Encode and decode

Encode file and store output in DER format
```shell
openssl@1.1 smime -encrypt -engine gost -binary -noattr -gost89 \
            -in input.txt -out output.enc -outform DER ca.cer
```

Decode file
```shell
openssl@1.1 smime -decrypt -engine gost -binary -noattr \
            -inkey ca.key -in output.enc -inform DER -out output.2.txt 
``` 

