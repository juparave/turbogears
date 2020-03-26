# Creating and installing SSL certificate

## Generate a CSR

Create a keypair

    $ openssl genrsa -out olinspections.com.key 2048

With the new key, sign a CSR

NOTE: Please do not enter your email address, challenge password or an optional company name when generating the CSR. Check [CSR](https://ssltools.digicert.com/checker/views/csrCheck.jsp)

    $ openssl genrsa -out olinspections.com.key 2048
    Country Name (2 letter code) [AU]:MX
    State or Province Name (full name) [Some-State]:Ciudad de Mexico
    Locality Name (eg, city) []:CDMX
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Online Inspections
    Organizational Unit Name (eg, section) []:Online Inspections Web Service
    Common Name (e.g. server FQDN or YOUR name) []:olinspections.com
    Email Address []:

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:

## Config

### Apache

    SSLEngine on
    #SSLProtocol all
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
    SSLHonorCipherOrder On
    Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
    Header always set X-Frame-Options DENY
    Header always set X-Content-Type-Options nosniff
    SSLCertificateFile /etc/apache2/ssl/cesarfer_com_ee.crt
    SSLCertificateKeyFile /etc/apache2/ssl/cesarfer.com.key
    SSLCACertificateFile /etc/apache2/ssl/cesarfer_com_apache.crt
    
    
