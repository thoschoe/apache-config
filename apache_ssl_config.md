# Apache Webserver SSL Konfiguration

Mozilla betreibt einen SSL-Konfigurator für verschiedene Server-Applikationen im Internet:  
https://ssl-config.mozilla.org/

Ubuntu/Debian Konfigurationsdatei:  
/etc/apache2/mods-available/ssl.conf  
Bei anderen Linux-Distributionen ggf. httpd.conf

Modul ssl muss aktiviert sein:  
```bash
a2enmod ssl
```
Es existiert noch das Modul gnutls, welches ebenfalls genutzt werden kann.  
Die Konfiguration hier ist abweichend.  
Im Beispiel wird das Modul ssl verwendet.

```apache
# TLSv1.2 Strong Ciphers ab 2019
SSLCipherSuite SSL HIGH:!aNULL:!ARIA:!CAMELLIA:!DHE:!AESCCM:!RSA:!SHA:!AES128+SHA256:!AES256+SHA384
# TLSv1.3
SSLCipherSuite TLSv1.3 TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256

SSLHonorCipherOrder on

SSLProtocol -all +TLSv1.2 +TLSv1.3
```
Hinweis:  
Der DNS-Name des Webservers muss dem Eintrag unter CommonName (CN) im Zertifikat entsprechen.
Das Zertifikat kann auch alternative DNS- bzw. IP-Einträge als so genannte SubjectAlternativeNames (SAN) enthalten.
Der CommonName kann auch Wildcards (*) enthalten (z. B. *.local.net).

Beispiel Webserver-Konfiguration

```apache
<VirtualHost *:443>

    ServerName webserver.local.net
    ServerAdmin webmaster@local.net

    # Log-Konfiguration
    # -----------------
    LogLevel info ssl:warn
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    ErrorLog  ${APACHE_LOG_DIR}/ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/ssl_access.log combined

    # SSL-Konfiguration
    # Modul ssl muss aktiviert sein (a2enmod ssl)
    # -----------------------------------------
    SSLEngine On

    # Zertifikat + Intermediate-Zertifikat + Diffie-Hellmann-Parameter
    SSLCertificateFile  /etc/apache2/ssl/webserver-zertifikatsdatei.crt
    SSLCertificateKeyFile /etc/apache2/ssl/webserver-zertifikat-private.key
 
    # Allgemeine Security Header
    # Modul headers muss aktiviert sein (a2enmod headers)
    # ---------------------------------------------------
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "sameorigin"
    Header always set X-XSS-Protection "1; mode=block"


</VirtualHost>

# Bsp. Uebergabe Passwort fuer SSL-Key falls vorhanden
SSLPassPhraseDialog exec:/etc/apache2/ssl/sslkeypasswd.sh
```