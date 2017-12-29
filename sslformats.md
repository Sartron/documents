# SSL Formats and Converting

## Introduction
This article will be covering how to convert an SSL certificate through various formats. Although not commonly needed, some clients may request to receive their SSL in a different format.  
There are also scenarios where software on a server may need to accept a different format than your typical PEM-encoded X.509 certificate. This is more commonly found on Windows systems.

## Using Online Tools
SSLShopper has a tool that you may use to convert certificate formats: [SSL Converter](https://www.sslshopper.com/ssl-converter.html)  
This tool will allow you to import the files for your SSL certificate and export them in the format you need.  
The provided page will also provide guidance on converting manually using OpenSSL. This article will be covering how to use OpenSSL for conversion below.

## Using OpenSSL
This section will cover certificate conversions using the `openssl` utility.
* If you are on Windows, you will require RDP access.
* If you are on Linux, you will require SSH access.
If you do not have access to the OpenSSL tool, you will have to use an online tool instead.

### PKCS#12/PFX
PKCS#12 files allow you to store multiple components needed for SSLs such as private keys, certificates, and CRLs. These types of files are more commonly used on Windows systems.  
Please note that you cannot create a PKCS#12 file using a PKCS#7 file as your input. An example on how to do this correctly will be provided below.  
The man page for pkcs12 provides more information than is present here in manipulating these files. If you are looking on how to extract certain components from it, I would recommend understanding how to use the various OpenSSL utilities with each other. This section will provide two examples to help illustrate this concept:
* Pipe the output of `openssl pkcs7` into `openssl pkcs12` to create a PKCS#12 file.
* Pipe the output of `openssl pkcs12` into `openssl rsa` to extract the RSA private key.

#### Basic Syntax
This section will only require use of the tool openssl pkcs12.
```
openssl pkcs12 [-export] [-chain] [-inkey filename] [-certfile filename] [-in filename] [-out filename]
```

#### Examples
* This is an example of creating a PKCS#12 container from the 3 components usually available in a typical SSL certificate installation. In the order they appear in, those components are the RSA private key, the certificate authority bundle, and the main SSL certificate which covers our domain name. After creating it, we can pump it back through `openssl pkcs12` in order to verify it is valid.
```
[user@server ~]# openssl pkcs12 -export -inkey ./inputkey -certfile ./inputbundle -in ./inputcert -out ./output.pfx
Enter Export Password:
Verifying - Enter Export Password:
[root@server ~]# openssl pkcs12 -in ./output.pfx -noout -info
Enter Import Password:
MAC Iteration 2048
MAC verified OK
PKCS7 Encrypted data: pbeWithSHA1And40BitRC2-CBC, Iteration 2048
Certificate bag
Certificate bag
Certificate bag
PKCS7 Data
Shrouded Keybag: pbeWithSHA1And3-KeyTripleDES-CBC, Iteration 2048
```
* You cannot pass a PKCS#7 file directly into `openssl pkcs12`. You must first pass it through `openssl pkcs7 -print_certs` and use that as your standard input instead. OpenSSL tools are generally able to handle extraneous details created by other OpenSSL utilities, which allow them to be piped into each other easily.
```
[user@server ~]# openssl pkcs12 -export -inkey ./key -out ./output.pfx < ./input.p7b
unable to load certificates
[user@server ~]# openssl pkcs7 -in ./input.p7b -print_certs | openssl pkcs12 -export -inkey ./key -out output.pfx
Enter Export Password:
Verifying - Enter Export Password:
[user@server ~]# openssl pkcs12 -in ./output.pfx -noout -info
Enter Import Password:
MAC Iteration 2048
MAC verified OK
PKCS7 Encrypted data: pbeWithSHA1And40BitRC2-CBC, Iteration 2048
Certificate bag
Certificate bag
Certificate bag
PKCS7 Data
Shrouded Keybag: pbeWithSHA1And3-KeyTripleDES-CBC, Iteration 2048
```
* Extract a private key from a PKCS#12 file: `openssl pkcs12 -in ./input.pfx -nocerts -nodes | openssl rsa`  
Please note that this you will be prompted to provide a password if the PKCS#12 file was exported with one, and another password if the private key within it is encrypted.

### PKCS#7
PKCS#7 files allow you to store multiple certificates within a single file.

#### Basic Syntax
This section will require the use of the tool `openssl crl2pkcs7` to convert our PEM certificates into the pkcs7 file, and the tool `openssl pkcs7` to parse our new file.
```
openssl crl2pkcs7 [-inform PEM|DER] [-outform PEM|DER] [-in filename] [-out filename] [-certfile filename] [-nocrl]
openssl pkcs7 [-inform PEM|DER] [-outform PEM|DER] [-in filename] [-out filename] [-print_certs] [-text]
```

#### Example
This is an example of creating a PKCS#7 container using the main certificate and certificate authority bundle. You may use -certfile to include any additional certificates as needed.  
I then parse the file by specify -print_certs -text. This is the equivalent of using openssl x509 -text.
```
[user@server ~]# openssl crl2pkcs7 -out ./output.p7b -certfile ./inputcert -certfile ./inputbundle -nocrl
[user@server ~]# openssl pkcs7 -in ./output.p7b -print_certs -text | grep -A4 'Certificate:'
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            83:54:d2:18:a2:83:85:30:ee:0b:fa:80:0f:26:6f:a9
--
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            f0:1d:4b:ee:7b:7c:a3:7b:3c:05:66:ac:05:97:24:58
--
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            27:66:ee:56:eb:49:f3:8e:ab:d7:70:a2:fc:84:de:22
```

### PEM/DER/NET
PEM, DER, and NET are the available forms of encoding an X.509 certificate using OpenSSL.
* PEM is also known as Base64 encoding.
* DER is also known as binary encoding.

#### Basic Syntax
This section will require the use of `openssl x509`. If you do not specify `-inform`, the tool will assume the input is PEM. This means if you are importing a PEM-encoded certificate, you do not need to specify this option.
```
openssl x509 [-inform DER|PEM|NET] [-outform DER|PEM|NET] [-in filename] [-out filename]
```

#### Example
This example will be converting a DER into a PEM certificate.
```
[user@server ~]# openssl x509 -in ./cert
unable to load certificate
140683010996128:error:0906D06C:PEM routines:PEM_read_bio:no start line:pem_lib.c:703:Expecting: TRUSTED CERTIFICATE
[user@server ~]# openssl x509 -inform DER -in ./cert -out ./cert
[user@server ~]# openssl x509 -in cert.pem -noout -text | grep -A4 'Certificate:'
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            83:54:d2:18:a2:83:85:30:ee:0b:fa:80:0f:26:6f:a9
```