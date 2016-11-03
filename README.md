# SpringBootDaplieSSL
Spring boot application with SSL communication using daple.com certificates

### Steps

1. Clone git clone from:- 

   git clone https://github.com/Daplie/localhost.daplie.com-certificates.git ./certs

   This will download all required certificates into <b>certs</b> folder
   
2. Following commands will create keystore out of certificates

   cat certs/chain.pem certs/root.pem > all.pem 

    openssl pkcs12 -export -chain -CAfile all.pem -in certs/cert.pem -inkey certs/privkey.pem -out localhostcert.keystore  -   name tomcat -passout pass:change
    
  The above commands <b>localhostcert.keystore</b> keystore, which will be used by the app.
  
3. Run the app and access the app via https://localhost.daplie.com:8443


