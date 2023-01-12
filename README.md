# OpenSSL_Certificate_Generation

## The script we will be using is ** mkcerts-modified.sh ** which is located in the public file repository of this project. The purpose of this script is to generate the required certs for the Cisco WLCs (Wireless LAN Controllers) and the gRPC Collectors.

## Before you proceed to generate the certificates with the script, make sure you are running OpenSSL 1.1.1 on your Ubuntu server. Why? Because OpenSSL 3.0 doesn’t generate the key as Private RSA Keys. This is the major difference between 3.0 and 1.1.1 versions. With OpenSSL 3.0 you need to specify which provider concept you want to use for your keys, if not, it will generate the standard generic keys and those keys will not work with your IOS-XE devices when you try to load the pem certs through the terminal. This is a workaround until we get the certs script working with OpenSSL 3.0.

How to know what version of OpenSSL you are running?
```bash
root@collector1:# openssl version
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
```
In case you are running OpenSSL 3.0 proceed to uninstall it:
```bash
root@collector1:# apt-get remove openssl -y
```
Then, proceed to install OpenSSL 1.1.1:
```bash
root@collector1:# wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb
root@collector1:# wget http://security.ubuntu.com/ubuntu/pool/main/o/openssl/openssl_1.1.1f-1ubuntu2.16_amd64.deb
root@collector1:# wget http://security.ubuntu.com/ubuntu/pool/main/o/openssl/libssl-dev_1.1.1f-1ubuntu2.16_amd64.deb

root@collector1:# sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb
root@collector1:# sudo dpkg -i libssl-dev_1.1.1f-1ubuntu2.16_amd64.deb
root@collector1:# sudo dpkg -i openssl_1.1.1f-1ubuntu2.16_amd64.deb

root@collector1:# openssl version
OpenSSL 1.1.1f  31 Mar 2020
```
After the installation of openssl is done you need to generate the certs with the mkcerts-modified.sh script, you can follow the steps provided below. The script will generate all the certificates needed for the collectors and the WLCs. In the below example we need to specify the FQDN of the Collectors and the hostname of the WLC.
```bash
root@collector1:~/example# ls
mkcerts-modified.sh

root@collector1:~/example# bash mkcerts-modified.sh collector1.yourdomainname.com wlc1.9840.yourdomainname.com
Configure trustpoints using the following:
crypto pki import <trustpoint name 1> pem terminal password admin12345
 <paste contents of collector1.yourdomainname.com-ca.crt>
 <paste contents of wlc1.9840.yourdomainname.com.key>
 <paste contents of wlc1.9840.yourdomainname.com-collector1.yourdomainname.com-ca.crt
crypto pki import <trustpoint name 2> pem terminal password admin12345
 <paste contents of wlc1.9840.yourdomainname.com-ca.crt>
 <paste contents of wlc1.9840.yourdomainname.com.key>
 <paste contents of wlc1.9840.yourdomainname.com-wlc1.9840.yourdomainname.com-ca.crt>

Configure telemetry using the following:

telemetry protocol grpc profile <profile name>
 ca-trustpoint <trustpoint name 1>
 id-trustpoint <trustpoint name 2>
telemetry receiver protocol <receiver name>
 host name collector1.amazonaccountteam.com <collector port>
 protocol grpc-tls profile <profile name>

root@collector1:~/example# ls
ca.cnf                                   collector1.amazonaccountteam.com.csr    wlc1.9840.amazonaccountteam.com.cnf
collector1.amazonaccountteam.com-ca.crt  collector1.amazonaccountteam.com.key    wlc1.9840.amazonaccountteam.com-collector1.amazonaccountteam.com-ca.crt
collector1.amazonaccountteam.com-ca.key  mkcerts-modified.sh                     wlc1.9840.amazonaccountteam.com.csr
collector1.amazonaccountteam.com-ca.srl  wlc1.9840.amazonaccountteam.com-ca.crt  wlc1.9840.amazonaccountteam.com.key
collector1.amazonaccountteam.com.cnf     wlc1.9840.amazonaccountteam.com-ca.key  wlc1.9840.amazonaccountteam.com-wlc1.9840.amazonaccountteam.com-ca.crt
collector1.amazonaccountteam.com.crt     wlc1.9840.amazonaccountteam.com-ca.srl
root@collector1:~/example# 
```
