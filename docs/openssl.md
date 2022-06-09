# Self signed certificate

    mkdir -p ca/{root-ca,sub-ca,server}/{private,certs,newcerts,crl,csr}
    touch ca/{root-ca,sub-ca}/index
    openssl rand -hex 16 > ca/root-ca/serial
    openssl rand -hex 16 > ca/sub-ca/serial

```
cd ca/server/private/
openssl req -new --newkey rsa:2048 -days 365 -nodes -x509 -subj '/CN=www.guillermososa.net' -keyout guillermososa.net.key -out guillermososa.net.crt
openssl x509 -text -in guillermososa.net.crt -noout
```
## Create the private key for root-ca,sub-ca,server

```bash
cd ca
openssl genrsa -aes256 -out root-ca/private/ca.key 4096
openssl genrsa -aes256 -out sub-ca/private/sub-ca.key 4096
openssl genrsa -out server/private/server.key 2048
```

## Create root-ca/root-ca.conf

```
cat > root-ca/root-ca.conf <<EOF
[ca]
#/root/ca/root-ca/root-ca.conf
#see man ca
default_ca              = CA_default

[CA_default]
dir                     = /root/ca/root-ca
certs                   = \$dir/certs
crl_dir                 = \$dir/crl
new_certs_dir           = \$dir/newcerts
database                = \$dir/index
serial                  = \$dir/serial
RANDFILE                = \$dir/private/.rand

private_key             = \$dir/private/ca.key
certificate             = \$dir/certs/ca.crt

crlnumber               = \$dir/crlnumber
crl                     = \$dir/crl/ca.crl
crl_extensions          = crl_ext
default_crl_days        = 30

default_md              = sha256

name_opt                = ca_default
cert_opt                = ca_default
default_days            = 365
preserve                = no
policy                  = policy_strict

[ policy_strict ]
countryName             = supplied
stateOrProvinceName     = supplied
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
#Options for the req tool, man req.
default_bits            = 2048
distinguished_name      = req_distinguished_name
string_mask             = utf8only
default_md              =  sha256
#Extension to add when the -x509 option is used.
x509_extensions         = v3_ca

[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address
countryName_default             = GB
stateOrProvinceName_default     = England
0.organizationName_default      = TheUrbanPenguin Ltd

[ v3_ca ]
#Extensions to apply when createing root ca
#Extensions for a typical CA, man x509v3_config
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid:always,issuer
basicConstraints                = critical, CA:true
keyUsage                        =  critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
#Extensions to apply when creating intermediate or sub-ca
#Extensions for a typical intermediate CA, same man as above
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid:always,issuer
#pathlen:0 ensures no more sub-ca can be created below an intermediate
basicConstraints                = critical, CA:true, pathlen:0
keyUsage                        = critical, digitalSignature, cRLSign, keyCertSign

[ server_cert ]
#Extensions for server certificates
basicConstraints                = CA:FALSE
nsCertType                      = server
nsComment                       =  "OpenSSL Generated Server Certificate"
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid,issuer:always
keyUsage                        =  critical, digitalSignature, keyEncipherment
extendedKeyUsage                = serverAuth

EOF
```

## Create the root certificate

```bash
cd root-ca/
openssl req -config root-ca.conf -key private/ca.key -new -x509 -days 7500 -sha256 -extensions v3_ca -out certs/ca.crt
openssl x509 -noout -in certs/ca.crt -text
```

## Create the intermediate certificate
```
cd ../sub-ca

cat > sub-ca.conf <<EOF
[ca]
#/root/ca/root-ca/root-ca.conf
#see man ca
default_ca              = CA_default

[CA_default]
dir                     = /root/ca/sub-ca
certs                   = \$dir/certs
crl_dir                 = \$dir/crl
new_certs_dir           = \$dir/newcerts
database                = \$dir/index
serial                  = \$dir/serial
RANDFILE                = \$dir/private/.rand

private_key             = \$dir/private/ca.key
certificate             = \$dir/certs/ca.crt

crlnumber               = \$dir/crlnumber
crl                     = \$dir/crl/ca.crl
crl_extensions          = crl_ext
default_crl_days        = 30

default_md              = sha256

name_opt                = ca_default
cert_opt                = ca_default
default_days            = 365
preserve                = no
policy                  = policy_strict

[ policy_strict ]
countryName             = supplied
stateOrProvinceName     = supplied
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
#Options for the req tool, man req.
default_bits            = 2048
distinguished_name      = req_distinguished_name
string_mask             = utf8only
default_md              =  sha256
#Extension to add when the -x509 option is used.
x509_extensions         = v3_ca

[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address
countryName_default             = GB
stateOrProvinceName_default     = England
0.organizationName_default      = TheUrbanPenguin Ltd

[ v3_ca ]
#Extensions to apply when createing root ca
#Extensions for a typical CA, man x509v3_config
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid:always,issuer
basicConstraints                = critical, CA:true
keyUsage                        =  critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
#Extensions to apply when creating intermediate or sub-ca
#Extensions for a typical intermediate CA, same man as above
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid:always,issuer
#pathlen:0 ensures no more sub-ca can be created below an intermediate
basicConstraints                = critical, CA:true, pathlen:0
keyUsage                        = critical, digitalSignature, cRLSign, keyCertSign

[ server_cert ]
#Extensions for server certificates
basicConstraints                = CA:FALSE
nsCertType                      = server
nsComment                       =  "OpenSSL Generated Server Certificate"
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid,issuer:always
keyUsage                        =  critical, digitalSignature, keyEncipherment
extendedKeyUsage                = serverAuth
subjectAltName                  = @alt_names

[alt_names]
DNS.1                           = guillermososa.net # Be sure to include the domain name here because Common Name is not so commonly honoured by itself
DNS.2                           = www.guillermososa.net # Optionally, add additional domains (I've added a subdomain here)
IP.1                            = 192.168.101.101 # Optionally, add an IP address (if the connection which you have planned requires it)

EOF
```

## create the certificate signed request for the sub-ca

 ```
openssl req -config sub-ca.conf -new -key private/sub-ca.key -sha256 -out csr/sub-ca.csr
 ```

### create the sub-ca certificate 

```
cd /root/ca/root-ca/
openssl ca -config root-ca.conf -extensions v3_intermediate_ca -days 3650 -notext -in ../sub-ca/csr/sub-ca.csr  -out ../sub-ca/certs/sub-ca.crt

openssl x509 -noout -text -in ../sub-ca/certs/sub-ca.crt
```


### Create server certificates
#### Create server csr         

```bash
cd /root/ca/server
openssl req -key private/server.key -new -sha256 -out csr/server.csr
```

#### create the signed certificate 

```bash
cd ../sub-ca/
openssl ca -config sub-ca.conf -extensions server_cert -days 365 -notext -in ../server/csr/server.csr -out ../server/certs/server.crt

cd ../server/certs/
#for nginx
cat server.crt ../../sub-ca/certs/sub-ca.crt > chained.crt
```
#### Test the server certificate
```bash
#add certificate to trusted
cp /root/ca/root-ca/certs/ca.crt /usr/local/share/ca-certificates/
update-ca-certificates

openssl s_server -accept 443 -www -key private/server.key -cert certs/server.crt -CAfile ../sub-ca/certs/sub-ca.crt
```
