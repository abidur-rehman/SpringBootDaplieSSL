Daplie is Taking Back the Internet!
--------------

[![](http://i.imgur.com/eG1lUZr.jpg)](https://daplie.com/preorder/)

Stop serving the empire and join the rebel alliance!

* [Invest in Daplie on Wefunder](https://daplie.com/invest/)
* [Pre-order Cloud](https://daplie.com/preorder/), The World's First Home Server for Everyone

# HTTPS certs for localhost development

HTTPS certificates for `localhost.daplie.com`, free for anyone to use in testing and development.

For the sake of keywords: most people (including myself) think of these as "SSL certificates" but they are, in fact, signed RSA keypairs used for TLS encryption.

## Install

```bash
# for use with any webserver
git clone https://github.com/Daplie/localhost.daplie.com-certificates.git ./certs

# as a node.js library
npm install --save localhost.daplie.com-certificates

# a quick and easy https server
npm install -g serve-https
```

```
├── privkey.pem           # private key in PEM format
├── cert.pem              # site certificate only
├── chain.pem             # intermetiate certificate only
└── fullchain.pem         # cert.pem + chain.pem
```

## Usage

QuickStart
----------

With bash

```bash
# serve https://localhost.daplie.com from current directory
serve-https

# serve from another directory and with an express app
serve-https -d /path/to/public/ --express-app /path/to/app.js
```

With https

```javascript
var https = require('https');
var httpsOptions = require('localhost.daplie.com-certificates').merge({});
var server = https.createServer(httpsOptions, function (req, res) {
  res.end("Hello, World!");
});

server.listen(443, function () {
  console.log("Ready and listening at");
  console.log("https://localhost.daplie.com:443/");
});
```

Or with `tls.createSecureContext`:

```javascript
var tls = require('tls');
var httpsOptions = require('localhost.daplie.com-certificates').merge({});
var tlsContext = tls.createSecureContext(httpsOptions);
```

API
---

* `merge(opts)` will merge our defaults into your opts object (preferring options you have set)
* `create(opts)` will create a new object with our defaults and your opts (preferring your options if both are set)

Our defaults:

```javascript
{
  key: '<<privkey.pem>>'
, cert: '<<cert.pem + chain.pem>>'            // for localhost.daplie.com
, ca: undefined
, crl: undefined
, requestCert: false
, rejectUnauthorized: true
, SNICallback: function (domainname, cb) {
    cb(null, secureContext);
  }
  , NPNProtocols: ['http/1.1']
}
```

## Manual Setup

If you've done this kind of thing before:

```bash
git clone https://github.com/Daplie/localhost.daplie.com-certificates.git ./certs
```

**Misnomer Alert**: Most webservers and software call for a **keypair** consisting of **server.crt** and **server.key**.
In most cases these actually correspond to **fullchain.pem** (crt) and **privkey.pem** (key).

<https://localhost.daplie.com> is an alias for <https://localhost> or <https://127.0.0.1>.

The benefit of using this certificate for localhost development is that you will have the exact same security policies
and APIs available in development as you would have in production.

### Let's Encrypt Certificate Conventions

The certificates are named according to the [Let's Encrypt](https://letsencrypt.org) conventions:

* privkey.pem - the server private key
* cert.pem - includes the bare server certficate only
* chain.pem - includes intermediate certificates only
* fullchain.pem - includes cert.pem and chain.pem
* root.pem - (proposed) includes any Root CAs

This convention is still subject to change.
See <https://github.com/letsencrypt/letsencrypt/issues/608>
and <https://groups.google.com/a/letsencrypt.org/forum/#!topic/client-dev/jE5uK4lPx5g>
to follow the conversation.

## Screencast + Article

[![screencast thumbnail](https://i.imgur.com/F8aoJg5.png)](https://youtu.be/r92gqYHJc5c)

[Create a CSR in PEM format for your HTTPS cert](https://coolaj86.com/articles/how-to-create-a-csr-for-https-tls-ssl-rsa-pems/)

[Examine HTTPS Certs with OpenSSL in Terminal](https://coolaj86.com/articles/how-to-examine-an-ssl-https-tls-cert/)

## Examples

### node.js

**Quick and Dirty:**

```bash
npm install --save-dev localhost.daplie.com-certificates
```

```javascript
'use strict';

var https = require('https');
var server = https.createServer(require('localhost.daplie.com-certificates').create());
var port = process.argv[2] || 8443;

server.on('request', function (req, res) {
  res.end('[' + req.method + ']' + ' ' + req.url);
});
server.listen(port, function () {
  console.log('Listening', server.address());
});
```

<https://localhost.daplie.com:8443/>

**DIY**

Instead of simply requiring `localhost.daplie.com-certificates` you will clone the certs yourself
and provide the options object.

```bash
git clone https://github.com/Daplie/localhost.daplie.com-certificates.git ./certs
```

```javascript
var fs = require('fs');
var path = require('path');
var certsPath = path.join(__dirname, 'certs');

//
// SSL Certificates
//
var options = {
  key: fs.readFileSync(path.join(certsPath, 'privkey.pem'), 'ascii')
, cert: fs.readFileSync(path.join(certsPath, 'fullchain.pem'), 'ascii')
/*
  // only for verification
, ca: [
    fs.readFileSync(path.join(certsPath, 'root.pem'))
  ]
, requestCert: true
*/
, rejectUnauthorized: true
, SNICallback: function (domainname, cb) {
    // normally we would check the domainname choose the correct certificate,
    // but for this demo we'll always use this one (the default) instead
    cb(null, require('tls').createSecureContext(options));
  }
, NPNProtcols: ['http/1.1']
};

var server = https.createServer(options);
```

### Caddy

* TODO


## How this was created

I created a directory `~/Code/localhost.daplie.com-certificates` (this repository, actually) and ran the following commands from that directory:



### 01 Create a Private Key

`01-create-key.sh`:
```bash
mkdir -p certs/server
openssl genrsa \
  -out certs/server/privkey.pem \
  2048
```

### 02 Create a Certificate Signing Request (CSR)

`02-create-csr.sh`:
```bash
mkdir -p certs/tmp
openssl req -new \
  -sha256 \
  -key certs/server/privkey.pem \
  -out certs/tmp/csr.pem \
  -subj "/C=US/ST=Utah/L=Provo/O=Daplie Inc/CN=localhost.daplie.com"
```

### 03 Copy and Paste the CSR to name.com's console

`cat certs/tmp/csr.pem`:
```
-----BEGIN CERTIFICATE REQUEST-----
MIICpTCCAY0CAQAwYDELMAkGA1UEBhMCVVMxDTALBgNVBAgTBFV0YWgxDjAMBgNV
BAcTBVByb3ZvMRMwEQYDVQQKEwpEYXBsaWUgSW5jMR0wGwYDVQQDExRsb2NhbGhv
c3QuZGFwbGllLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJ3j
1nY+5bJf2oWVRGCrwTQ7Mw/qzMMu62RgGZawN2d6QTDSYBSCZdyuEpwOiDy6AO9x
Wqo6WJx7yu6Yv04syZEbc5tLMtX77YROAF7GkRrBIkqPtSkKnDYQm0wW9I9escgy
GQ3itSSHU/Oijv6Lj8xUigM+WM+DE860U1K0QID/eQPYOWQhj/A6WQXxPWWDsDxD
3ZpVeLIgeZe5usd1PhuGvhhFvK+W0QHZ4D7PgsvKrP6Qwoc3VNiEwlQa6v8L8t7e
w2uEXa96o4J08GiZPClbAng8+Y3SSp5PQ3cPUIlWu3hSPxb03t8+yC5gB6Gzl7To
wJwBPcOXUSo00QnD96UCAwEAAaAAMA0GCSqGSIb3DQEBCwUAA4IBAQCP04HRbk1x
i9ESsClWoClyG8VZCPGcG2KooQ2tqKaCBRGG9hNz1vm1SzUyclKz1CMZgI5i+b02
h/zeJRHkQ9ztT07oRUmKK1/tDt88J3AH3wIcnMEyzT3kHRuJbrZ81hEz417tePhs
v4/NziQc8Xv8WJP6sjcg72L5jlV0qrc3BYkdOqgjIOMOJoo7pNCbmh0xCvW5FURc
uG1AUaFPaDcOshT3YOlH9MP5/SoYl5X8y1SJVbNDOrQzJo8Erw1HoxOX4tRTd3F+
BalBlrLZQMvgtOkMNErebgARAz6xlfzXpOf7G0AkvllHJAnzTmSalzR5hDWdfcbq
mnxzBDw4+wI+
-----END CERTIFICATE REQUEST-----
```

### 04 Follow Validation Procedure

I bought the domain on name.com so I could have used the automatic validation process,
but since I have my GLUE records and DNS management for the daplie.com DNS elsewhere and
I didn't want to go through the hassle of the validation records, I used the registered admin
email address (which I happened to already have setup through mailgun).

This is the email I got:

```
ORDER APPROVAL

Dear Domain Administrator,

You are receiving this email because you are the Domain Administrator for localhost.daplie.com and the person identified below has requested a RapidSSL certificate for:

localhost.daplie.com


Applicant Information:
     Name:   AJ ONeal
     Email:  coolaj86@gmail.com
     Phone:  +1.3174266525

AJ ONeal requests that you come to the URL below to review and approve this certificate request:

     https://products.geotrust.com/orders/A.do?p=Ac8lMXMpxHsbZVlWJwBcF

Please follow the above link and click either the I APPROVE or I DO NOT APPROVE button.

When you click I APPROVE the certificate will be issued and emailed to the Applicant, Approver, and Technical contacts.

If you click I DO NOT APPROVE the certificate application will be cancelled.

Thanks,

RapidSSL Customer Support
http://www.rapidssl.com/support
Hours of Operation: Mon - Fri 09:00 - 17:00 (EST)
Email:     orderprocessing@rapidssl.com
Live Chat: https://knowledge.rapidssl.com/support/ssl-certificate-support/index.html
```

And once I clicked the link, this was the confirmation email I got back:

```
Dear AJ ONeal,

Congratulations! RapidSSL has approved your request for a RapidSSL certificate. Your certificate is included at the end of this email.

INSTALLATION INSTRUCTIONS

1. INSTALL CERTIFICATE:
Install the X.509 version of your certificate included at the end of this e-mail.
For installation instructions for your SSL Certificate, go to:
https://knowledge.rapidssl.com/support/ssl-certificate-support/index?page=content&id=SO16226

2. INTERMEDIATE CERTIFICATE ADVISORY:
You MUST install the RapidSSL intermediate Certificate on your server together with your Certificate or it may not operate correctly.

** MICROSOFT IIS and TOMCAT USERS
Microsoft and Tomcat users are advised to download a PKCS #7 formatted certificate from the GeoTrust User Portal:
https://products.geotrust.com/orders/orderinformation/authentication.do. PKCS #7 is the default format used by these vendors during installation and includes the intermediate CA certificate.

You can get your RapidSSL Intermediate Certificates at:
https://knowledge.rapidssl.com/support/ssl-certificate-support/index?page=content&id=AR1548

3. CHECK INSTALLATION:
Ensure you have installed your certificate correctly at:
https://knowledge.rapidssl.com/support/ssl-certificate-support/index?page=content&id=AR1549

4. INSTALL THE RAPIDSSL SITE SEAL:
Additionally, as part of your SSL Certificate Service, you are entitled to display the RapidSSL Site Seal - recognized across the Internet and around the world as a symbol of authenticity, security, and trust - to build consumer confidence in your Web site.

Installation instructions for the RapidSSL Site Seal can be found on the following link:
https://knowledge.rapidssl.com/support/ssl-certificate-support/index?page=content&id=SO14424&actp=LIST&viewlocale=en_US

If you require additional technical support please contact Name.com.

Web Server CERTIFICATE
-----------------

-----BEGIN CERTIFICATE-----
MIIEqzCCA5OgAwIBAgIDBPiXMA0GCSqGSIb3DQEBCwUAMEcxCzAJBgNVBAYTAlVT
MRYwFAYDVQQKEw1HZW9UcnVzdCBJbmMuMSAwHgYDVQQDExdSYXBpZFNTTCBTSEEy
NTYgQ0EgLSBHMzAeFw0xNTA2MDkwOTI3NDJaFw0xNjA2MTEyMDA4MDdaMIGYMRMw
EQYDVQQLEwpHVDcyMjM4NTY5MTEwLwYDVQQLEyhTZWUgd3d3LnJhcGlkc3NsLmNv
bS9yZXNvdXJjZXMvY3BzIChjKTE1MS8wLQYDVQQLEyZEb21haW4gQ29udHJvbCBW
YWxpZGF0ZWQgLSBSYXBpZFNTTChSKTEdMBsGA1UEAxMUbG9jYWxob3N0LmRhcGxp
ZS5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCd49Z2PuWyX9qF
lURgq8E0OzMP6szDLutkYBmWsDdnekEw0mAUgmXcrhKcDog8ugDvcVqqOlice8ru
mL9OLMmRG3ObSzLV++2ETgBexpEawSJKj7UpCpw2EJtMFvSPXrHIMhkN4rUkh1Pz
oo7+i4/MVIoDPljPgxPOtFNStECA/3kD2DlkIY/wOlkF8T1lg7A8Q92aVXiyIHmX
ubrHdT4bhr4YRbyvltEB2eA+z4LLyqz+kMKHN1TYhMJUGur/C/Le3sNrhF2veqOC
dPBomTwpWwJ4PPmN0kqeT0N3D1CJVrt4Uj8W9N7fPsguYAehs5e06MCcAT3Dl1Eq
NNEJw/elAgMBAAGjggFMMIIBSDAfBgNVHSMEGDAWgBTDnPP800YINLvORn+gfFvz
4gjLWTBXBggrBgEFBQcBAQRLMEkwHwYIKwYBBQUHMAGGE2h0dHA6Ly9ndi5zeW1j
ZC5jb20wJgYIKwYBBQUHMAKGGmh0dHA6Ly9ndi5zeW1jYi5jb20vZ3YuY3J0MA4G
A1UdDwEB/wQEAwIFoDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwHwYD
VR0RBBgwFoIUbG9jYWxob3N0LmRhcGxpZS5jb20wKwYDVR0fBCQwIjAgoB6gHIYa
aHR0cDovL2d2LnN5bWNiLmNvbS9ndi5jcmwwDAYDVR0TAQH/BAIwADBBBgNVHSAE
OjA4MDYGBmeBDAECATAsMCoGCCsGAQUFBwIBFh5odHRwczovL3d3dy5yYXBpZHNz
bC5jb20vbGVnYWwwDQYJKoZIhvcNAQELBQADggEBAGlPWTo4Z7oS6E5QPVhFr0kH
wdyGqFD3u93Nxa9L2Hfs2UrpJhhrliux/C9mxgk1O1bgVGhVvQNhiTUBSkJaIMCQ
aG5cQPBLV5u+vK+YFJHK8F+C0/vKU/xcEp4Ae1JNkIoXnfdPbGGbIS82HYp2uveD
dtv5/hqIdLfT6TRFZ7IbhCvTR0iYzPRsOB68PSWKHyVcolK2EHIHdo7Zjs/0tEF5
+4g/NKqX7zAMtMwQ9puPxm6M4BDnJjfiicH+4SeaRG72qpV56mHAeEOeIB4WQ61d
QyTmfubJfT/S1IBFfwqLln/Kf3PGyOvoOYocFpkfHvzFrviqljDDIyfVWx7hQpE=
-----END CERTIFICATE-----
```

### 05 Create Files from the provided Certificate (and intermediates)

#### Domain Name

`localhost.daplie.com`

#### Server Certificate

`cert.pem`:
```
-----BEGIN CERTIFICATE-----
MIIFajCCBFKgAwIBAgIQdsKX6kswrkbK7NZ/kc31vTANBgkqhkiG9w0BAQsFADBC
MQswCQYDVQQGEwJVUzEWMBQGA1UEChMNR2VvVHJ1c3QgSW5jLjEbMBkGA1UEAxMS
UmFwaWRTU0wgU0hBMjU2IENBMB4XDTE2MDYxMDAwMDAwMFoXDTE3MDcxMDIzNTk1
OVowHzEdMBsGA1UEAwwUbG9jYWxob3N0LmRhcGxpZS5jb20wggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQCd49Z2PuWyX9qFlURgq8E0OzMP6szDLutkYBmW
sDdnekEw0mAUgmXcrhKcDog8ugDvcVqqOlice8rumL9OLMmRG3ObSzLV++2ETgBe
xpEawSJKj7UpCpw2EJtMFvSPXrHIMhkN4rUkh1Pzoo7+i4/MVIoDPljPgxPOtFNS
tECA/3kD2DlkIY/wOlkF8T1lg7A8Q92aVXiyIHmXubrHdT4bhr4YRbyvltEB2eA+
z4LLyqz+kMKHN1TYhMJUGur/C/Le3sNrhF2veqOCdPBomTwpWwJ4PPmN0kqeT0N3
D1CJVrt4Uj8W9N7fPsguYAehs5e06MCcAT3Dl1EqNNEJw/elAgMBAAGjggJ9MIIC
eTAfBgNVHREEGDAWghRsb2NhbGhvc3QuZGFwbGllLmNvbTAJBgNVHRMEAjAAMCsG
A1UdHwQkMCIwIKAeoByGGmh0dHA6Ly9ncC5zeW1jYi5jb20vZ3AuY3JsMG8GA1Ud
IARoMGYwZAYGZ4EMAQIBMFowKgYIKwYBBQUHAgEWHmh0dHBzOi8vd3d3LnJhcGlk
c3NsLmNvbS9sZWdhbDAsBggrBgEFBQcCAjAgDB5odHRwczovL3d3dy5yYXBpZHNz
bC5jb20vbGVnYWwwHwYDVR0jBBgwFoAUl8InUJ7CyewMiDLIfK3ipgFP2m8wDgYD
VR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjBXBggr
BgEFBQcBAQRLMEkwHwYIKwYBBQUHMAGGE2h0dHA6Ly9ncC5zeW1jZC5jb20wJgYI
KwYBBQUHMAKGGmh0dHA6Ly9ncC5zeW1jYi5jb20vZ3AuY3J0MIIBAgYKKwYBBAHW
eQIEAgSB8wSB8ADuAHYA3esdK3oNT6Ygi4GtgWhwfi6OnQHVXIiNPRHEzbbsvswA
AAFVOonOzQAABAMARzBFAiEAzh4K7ZOSGCCFFvzAvrfl+o5AKcnmV7NHPgQZe3x4
hZgCIH/M2LZI1OSdkQbF2wgD/xH4PvQ4i8TTOdGB0WAYVr1eAHQApLkJkLQYWBSH
uxOizGdwCjw1mAT5G9+443fNDsgN3BAAAAFVOonO8gAABAMARTBDAh84cfLQthSR
Pe6hFgL8TPSuCUxIFBcEbnIPNB7ZxQwYAiBTClbmIn81bBkwAjasJu2u+UdxGE0i
Wx5lFe5X9pqsUTANBgkqhkiG9w0BAQsFAAOCAQEAAWYuT/fTBZdXb4kwoVaUnc82
2CEnGuOHr9QMdGRMqWJRe068StADdw1u3V6bcB7+mBiGl8C+WOLhv9WxYKqNFvyj
Eeaeekb4GqfrfuxNvoOU/vHdYaww2J9N1ESgIV4BdFF8aNgOnjpRcKSMsMgzNJdU
lh6l7jhnTeNYCyMnn+2dVQBcRQvptKmpkS4sK6NAVSMWDioImEoGj0PCdLqG8k21
d3vNddCEQmcNUTHs38nswUKZxfQKpjo+z9jBFmurmaNqSFnd8ySmBELZjXEOXEQz
KBlUSDj3UYVmH49t0toGyHVfKHPCBLyUZhvUTy0tNVgn9Nc+/MXJnv+c5rxVeQ==
-----END CERTIFICATE-----
```

#### CA Certificates

#### INTERMEDIATE

`intermediate.crt.pem`:
```
-----BEGIN CERTIFICATE-----
MIIETTCCAzWgAwIBAgIDAjpxMA0GCSqGSIb3DQEBCwUAMEIxCzAJBgNVBAYTAlVTMRYwFAYDVQQK
Ew1HZW9UcnVzdCBJbmMuMRswGQYDVQQDExJHZW9UcnVzdCBHbG9iYWwgQ0EwHhcNMTMxMjExMjM0
NTUxWhcNMjIwNTIwMjM0NTUxWjBCMQswCQYDVQQGEwJVUzEWMBQGA1UEChMNR2VvVHJ1c3QgSW5j
LjEbMBkGA1UEAxMSUmFwaWRTU0wgU0hBMjU2IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
CgKCAQEAu1jBEgEul9h9GKrIwuWF4hdsYC7JjTEFORoGmFbdVNcRjFlbPbFUrkshhTIWX1SG5tmx
2GCJa1i+ctqgAEJ2sSdZTM3jutRc2aZ/uyt11UZEvexAXFm33Vmf8Wr3BvzWLxmKlRK6msrVMNI4
/Bk7WxU7NtBDTdFlodSLwWBBs9ZwF8w5wJwMoD23ESJOztmpetIqYpygC04q18NhWoXdXBC5VD0t
A/hJ8LySt7ecMcfpuKqCCwW5Mc0IW7siC/acjopVHHZDdvDibvDfqCl158ikh4tq8bsIyTYYZe5Q
Q7hdctUoOeFTPiUs2itP3YqeUFDgb5rE1RkmiQF1cwmbOwIDAQABo4IBSjCCAUYwHwYDVR0jBBgw
FoAUwHqYaI2J+6sFZAwRfap9ZbjKzE4wHQYDVR0OBBYEFJfCJ1CewsnsDIgyyHyt4qYBT9pvMBIG
A1UdEwEB/wQIMAYBAf8CAQAwDgYDVR0PAQH/BAQDAgEGMDYGA1UdHwQvMC0wK6ApoCeGJWh0dHA6
Ly9nMS5zeW1jYi5jb20vY3Jscy9ndGdsb2JhbC5jcmwwLwYIKwYBBQUHAQEEIzAhMB8GCCsGAQUF
BzABhhNodHRwOi8vZzIuc3ltY2IuY29tMEwGA1UdIARFMEMwQQYKYIZIAYb4RQEHNjAzMDEGCCsG
AQUFBwIBFiVodHRwOi8vd3d3Lmdlb3RydXN0LmNvbS9yZXNvdXJjZXMvY3BzMCkGA1UdEQQiMCCk
HjAcMRowGAYDVQQDExFTeW1hbnRlY1BLSS0xLTU2OTANBgkqhkiG9w0BAQsFAAOCAQEANevhiyBW
lLp6vXmp9uP+bji0MsGj21hWID59xzqxZ2nVeRQb9vrsYPJ5zQoMYIp0TKOTKqDwUX/N6fmS/Zar
RfViPT9gRlATPSATGC6URq7VIf5Dockj/lPEvxrYrDrK3maXI67T30pNcx9vMaJRBBZqAOv5jUOB
8FChH6bKOvMoPF9RrNcKRXdLDlJiG9g4UaCSLT+Qbsh+QJ8gRhVd4FB84XavXu0R0y8TubglpK9Y
Ca81tGJUheNI3rzSkHp6pIQNo0LyUcDUrVNlXWz4Px8G8k/Ll6BKWcZ40egDuYVtLLrhX7atKz4l
ecWLVtXjCYDqwSfC2Q7sRwrp0Mr82A==
-----END CERTIFICATE-----
```

#### ROOT

`root.crt.pem`:
```
-----BEGIN CERTIFICATE-----
MIIDVDCCAjygAwIBAgIDAjRWMA0GCSqGSIb3DQEBBQUAMEIxCzAJBgNVBAYTAlVT
MRYwFAYDVQQKEw1HZW9UcnVzdCBJbmMuMRswGQYDVQQDExJHZW9UcnVzdCBHbG9i
YWwgQ0EwHhcNMDIwNTIxMDQwMDAwWhcNMjIwNTIxMDQwMDAwWjBCMQswCQYDVQQG
EwJVUzEWMBQGA1UEChMNR2VvVHJ1c3QgSW5jLjEbMBkGA1UEAxMSR2VvVHJ1c3Qg
R2xvYmFsIENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2swYYzD9
9BcjGlZ+W988bDjkcbd4kdS8odhM+KhDtgPpTSEHCIjaWC9mOSm9BXiLnTjoBbdq
fnGk5sRgprDvgOSJKA+eJdbtg/OtppHHmMlCGDUUna2YRpIuT8rxh0PBFpVXLVDv
iS2Aelet8u5fa9IAjbkU+BQVNdnARqN7csiRv8lVK83Qlz6cJmTM386DGXHKTubU
1XupGc1V3sjs0l44U+VcT4wt/lAjNvxm5suOpDkZALeVAjmRCw7+OC7RHQWa9k0+
bw8HHa8sHo9gOeL6NlMTOdReJivbPagUvTLrGAMoUgRx5aszPeE4uwc2hGKceeoW
MPRfwCvocWvk+QIDAQABo1MwUTAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBTA
ephojYn7qwVkDBF9qn1luMrMTjAfBgNVHSMEGDAWgBTAephojYn7qwVkDBF9qn1l
uMrMTjANBgkqhkiG9w0BAQUFAAOCAQEANeMpauUvXVSOKVCUn5kaFOSPeCpilKIn
Z57QzxpeR+nBsqTP3UEaBU6bS+5Kb1VSsyShNwrrZHYqLizz/Tt1kL/6cdjHPTfS
tQWVYrmm3ok9Nns4d0iXrKYgjy6myQzCsplFAMfOEVEiIuCl6rYVSAlk6l5PdPcF
PseKUgzbFbS9bZvlxrFUaKnjaZC2mqUPuLk/IH2uSrW4nOQdtqvmlKXBx4Ot2/Un
hw4EbNX/3aBd7YdStysVAq45pmp06drE57xNNB6pXE0zX5IJL4hmXXeXxx12E6nV
5fEWCRE11azbJHFwLJhWC9kXtNHjUStedejV0NxPNO3CBWaAocvmMw==
-----END CERTIFICATE-----
```

### 06 Bundle the certificates (for Caddy et al)

```bash
cat server/privkey.pem > privkey.pem

cat server/cert.pem > cert.pem

cat ca/intermediate.crt.pem > chain.pem

cat server/cert.pem ca/intermediate.crt.pem > fullchain.pem

cat server/ca.crt.pem > root.pem
```

**Note**: The order *may* be important. It *should* be from least to greatest authority as seen above.
