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
