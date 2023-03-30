# OpenSSL_Certificate_Generation

## The script we will be using is ** mkcerts-modified.sh ** which is located in the public file repository of this project. The purpose of this script is to generate the required certs for the Cisco WLCs (Wireless LAN Controllers) and the gRPC Collectors.

## Before you proceed to generate the certificates with the script, make sure you are running OpenSSL 1.1.1 on your Ubuntu server. Why? Because OpenSSL 3.0 doesn’t generate the key as Private RSA Keys. This is the major difference between 3.0 and 1.1.1 versions. With OpenSSL 3.0 you need to specify which provider concept you want to use for your keys, if not, it will generate the standard generic keys and those keys will not work with your IOS-XE devices when you try to load the pem certs through the terminal. This is a workaround until we get the certs script working with OpenSSL 3.0.

Also, please make sure you sync-up all your devices with a NTP Server to avoid issues with different timezones and certificate to be in sync. Certificates are backdated by one hour to allow for clock skew. All certificates use UTC time, the time zone is stored as part of the date in the certificate. User agents in other timezones are fine as long as their time zone and clocks is configured correctly.

This process is to generate the certificates needed for gRPC mTLS authentication.

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
When the installation of openssl is done you need to generate the certs with the mkcerts-modified.sh script, you can follow the steps provided below. The script will generate all the certificates needed for the collectors and the WLCs. In the below example we need to specify the FQDN of the Collectors and the hostname of the WLC.
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
Here is how you load the Certs after generated to the WLCs for mTLS authentication:
1st cert:
```bash
wlc1.9840#crypto pki import ca1 pem terminal password admin12345

% Enter PEM-formatted CA certificate.
% End with a blank line or "quit" on a line by itself.
-----BEGIN CERTIFICATE-----
       cert-content
-----END CERTIFICATE-----
quit
% Enter PEM-formatted encrypted private General Purpose key.
% End with "quit" on a line by itself.
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,XXXXXXXXXXXXX
-----BEGIN CERTIFICATE-----
       cert-content
-----END CERTIFICATE----- 
quit
% Enter PEM-formatted General Purpose certificate.
% End with a blank line or "quit" on a line by itself.
-----BEGIN CERTIFICATE-----
       cert-content 
-----END CERTIFICATE-----
quit
% PEM files import succeeded.
```
2nd cert:
```bash
wlc1.9840#crypto pki import id1 pem terminal password admin12345 

% Enter PEM-formatted CA certificate.
% End with a blank line or "quit" on a line by itself.
-----BEGIN CERTIFICATE----- 
       cert-content 
-----END CERTIFICATE----- 
quit
% Enter PEM-formatted encrypted private General Purpose key.
% End with "quit" on a line by itself.
-----BEGIN RSA PRIVATE KEY----- 
Proc-Type: 4,ENCRYPTED 
DEK-Info: DES-EDE3-CBC,XXXXXXXXXXXXX 
-----BEGIN CERTIFICATE----- 
       cert-content  
-----END CERTIFICATE----- 
quit 
% Enter PEM-formatted General Purpose certificate.
% End with a blank line or "quit" on a line by itself.
-----BEGIN CERTIFICATE----- 
       cert-content 
-----END CERTIFICATE----- 
quit 
% PEM files import succeeded.
```
Telemetry CLI configuration setup on the WLC with the certificates and setup of an xpath:
```bash
telemetry protocol grpc profile mtlsyangsuite
 ca-trustpoint ca1
 id-trustpoint id1

telemetry receiver protocol mtlsyangsuite
 host name collector1.yourdomainname.com 57500
 protocol grpc-tls profile mtlsyangsuite

telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
 receiver-type protocol
 source-address 10.93.178.68
 stream yang-push
 update-policy periodic 1000
 receiver name mtlsyangsuite
```
After the setup of the collectors is done and the WLC as well, you should see everything up and running on the WLC:
```bash
wlc1.9840#show telemetry connection all
Telemetry connections

Index Peer Address               Port  VRF Source Address             State      State Description
----- -------------------------- ----- --- -------------------------- ---------- --------------------
  401 10.93.178.143              57500 0   10.93.178.68               Active     Connection up       
  402 10.93.178.142              57500 0   10.93.178.68               Active     Connection up       
```
