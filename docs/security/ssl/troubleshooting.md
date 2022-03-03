# SSL Troubleshooting

## Show certificate for a host

`openssl s_client -showcerts -connect www.domain.com:443`

Output:
```
CONNECTED(00000003)
--snip--
---
Certificate chain
 0 s:/C=US/ST=Texas/L=Carrollton/O=Woot Inc/CN=*.woot.com
   i:/C=US/O=SecureTrust Corporation/CN=SecureTrust CA
-----BEGIN CERTIFICATE-----
--snip--
-----END CERTIFICATE-----
 1 s:/C=US/O=SecureTrust Corporation/CN=SecureTrust CA
   i:/C=US/O=Entrust.net/OU=www.entrust.net/CPS incorp. by ref. (limits liab.)/OU=(c) 1999 Entrust.net Limited/CN=Entrust.net Secure Server Certification Authority
-----BEGIN CERTIFICATE-----
--snip--
-----END CERTIFICATE-----
---
Server certificate
subject=/C=US/ST=Texas/L=Carrollton/O=Woot Inc/CN=*.woot.com
issuer=/C=US/O=SecureTrust Corporation/CN=SecureTrust CA
---
No client certificate CA names sent
---
SSL handshake has read 2123 bytes and written 300 bytes
---
New, TLSv1/SSLv3, Cipher is RC4-MD5
Server public key is 1024 bit
--snip--
```

*([Source](https://langui.sh/2009/03/14/checking-a-remote-certificate-chain-with-openssl/))*

## Check the expiration date of an SSL certificate

`openssl s_client -servername <NAME> -connect <HOST:PORT> 2>/dev/null | openssl x509 -noout -dates`

Output:
```
$ echo q | openssl s_client -showcerts -connect control.akamai.com:443 -servername control.akamai.com 2>/dev/null | openssl x509 -noout -dates
notBefore=Sep 23 00:00:00 2021 GMT
notAfter=Sep 22 23:59:59 2022 GMT
```

*([Source](https://learn.akamai.com/en-us/webhelp/enterprise-application-access/enterprise-application-access/GUID-9D88336D-2733-4325-913C-916403E03D48.html))*

## Some snippets

But what if you want to connect to something other than a bog standard webserver on port 443? Well, if you need to use starttls that is also available. As of OpenSSL 0.9.8 you can choose from smtp, pop3, imap, and ftp as starttls options.

`openssl s_client -showcerts -starttls imap -connect mail.domain.com:139`

If you need to check using a specific SSL version (perhaps to verify if that method is available) you can do that as well. -ssl2, -ssl3, -tls1, and -dtls1 are all choices here.2

`openssl s_client -showcerts -ssl2 -connect www.domain.com:443`

You can also present a client certificate if you are attempting to debug issues with a connection that requires one.3

`openssl s_client -showcerts -cert cert.cer -key cert.key -connect www.domain.com:443`

And for those who really enjoy playing with SSL handshakes, you can even specify acceptable ciphers.4

`openssl s_client -showcerts -cipher DHE-RSA-AES256-SHA -connect www.domain.com:443`

*([Source](https://langui.sh/2009/03/14/checking-a-remote-certificate-chain-with-openssl/))*



More: 

***

	
	$ curl --insecure -vvI https://www.google.com 2>&1 | awk 'BEGIN { cert=0 } /^\* SSL connection/ { cert=1 } /^\*/ { if (cert) print }'

	* SSL connection using TLSv1.2 / ECDHE-ECDSA-CHACHA20-POLY1305
	* ALPN, server accepted to use h2
	* Server certificate:
	*  subject: C=US; ST=California; L=Mountain View; O=Google LLC; CN=www.google.com
	*  start date: Jul  7 08:10:21 2020 GMT
	*  expire date: Sep 29 08:10:21 2020 GMT
	*  issuer: C=US; O=Google Trust Services; CN=GTS CA 1O1
	*  SSL certificate verify ok.
	* Using HTTP2, server supports multi-use
	* Connection state changed (HTTP/2 confirmed)
	* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
	* Using Stream ID: 1 (easy handle 0x7ff34f809e00)
	* Connection state changed (MAX_CONCURRENT_STREAMS == 100)!
	* Connection #0 to host www.google.com left intact
	* Closing connection 0


	
	$ openssl s_client -connect www.google.com:https
	CONNECTED(00000005)
	depth=2 OU = GlobalSign Root CA - R2, O = GlobalSign, CN = GlobalSign
	verify return:1
	depth=1 C = US, O = Google Trust Services, CN = GTS CA 1O1
	verify return:1
	depth=0 C = US, ST = California, L = Mountain View, O = Google LLC, CN = www.google.com
	verify return:1
	---
	Certificate chain
	 0 s:/C=US/ST=California/L=Mountain View/O=Google LLC/CN=www.google.com
	   i:/C=US/O=Google Trust Services/CN=GTS CA 1O1
	 1 s:/C=US/O=Google Trust Services/CN=GTS CA 1O1
	   i:/OU=GlobalSign Root CA - R2/O=GlobalSign/CN=GlobalSign
	---
	Server certificate
	-----BEGIN CERTIFICATE-----
	MIIFkTCCBHmgAwIBAgIQNqmF7X67WckCAAAAAHJrQTANBgkqhkiG9w0BAQsFADBC
	MQswCQYDVQQGEwJVUzEeMBwGA1UEChMVR29vZ2xlIFRydXN0IFNlcnZpY2VzMRMw
	EQYDVQQDEwpHVFMgQ0EgMU8xMB4XDTIwMDcwNzA3NTg0OVoXDTIwMDkyOTA3NTg0
	[...]

