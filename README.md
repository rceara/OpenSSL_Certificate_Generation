# OpenSSL_Certificate_Generation

## The script we will be using is ** mkcerts-modified.sh** which is located in the public file repository of this project.

## Before you proceed to generate the certificates with the script, make sure you are running OpenSSL 1.1.1 on your Ubuntu server. Why? Because OpenSSL 3.0 doesn’t generate the key as Private RSA Keys. This is the major difference between 3.0 and 1.1.1 versions. With OpenSSL 3.0 you need to specify which provider concept you want to use for your keys, if not, it will generate the standard generic keys and those keys will not work with your IOS-XE devices when you try to load the pem certs through the terminal. This is a workaround until we get the certs script working with OpenSSL 3.0.

