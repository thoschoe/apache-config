# Apache Reverse Proxy Konfiguration

Ein Reverse-Proxy Server wird in der Regel vor einen oder meherere Web-Anwendungsserver geschaltet.

Module proxy proxy_http und proxy_ajp aktivieren:

```bash
a2enmod proxy proxy_http proxy_ajp
```

Es können DNS-Namen (Hostnamen) oder auch IP-Adressen verwendet werden.
(Bei HTTPS muss der Host-Name bzw. die IP im CommonName Eintrag oder als
SubjectAlternativeName im Zertifikat des Applikationsservers gesetzt sein.)

Bei den Standardprotokollen http und https muss kein Port angegeben werden,
wenn die Standardports 80 bzw. 443 verwendet werden.

Der Zugriff auf Java-Anwendungsserver (z. B. Tomcat, JBoss/Wildfly) kann per AJP
(Apache JServ Protokoll) erfolgen. Aus Sicherheitsgründen sollte hier ein Secret
verwendet werden. Das Secret in der AJP-Connector-Konfiguration des Anwendungsservers
auch verwendet werden. Der AJP Standard-Port ist 8009 TCP.
(siehe auch: https://de.wikipedia.org/wiki/Apache_JServ_Protocol)

### Beispielkonfiguration

```apache
<VirtualHost *443>
# Webserver SSL-Konfiguration siehe apache_ssl_config.md

# Forward-Proxy-Funktion deaktivieren
ProxyRequests Off

# ggf. Timeout zum Applikaitonsserver hinter dem Reverse Proxy
ProxyTimeout 600

# ggf. Slash-Encoding z. B. fuer GitLab-Api
# / wird als %2F in der URL verwendet und soll nicht ersetzt werden
AllowEncodedSlashes NoDecode

# Konfiguration für HTTPS-Zugriff zum Anwendungsserver hinter dem ReverseProxy
# ============================================================================

# Trusted CA-Zertifikat (wenn eigene CA im Unternehmen vorhanden ist)
SSLCACertificateFile /etc/apache2/ssl/eigene-ca.crt

# Proxy-SSL-Einstellungen für HTTPS-Zugriff zum Applikaitonsserver
SSLProxyEngine On
SSLProxyVerify require
SSLProxyCheckPeerCN On
SSLProxyCheckPeerName On
SSLProxyCheckPeerExpire On
SSLProxyCACertificateFile /etc/apache2/ssl/root-ca-zertifikat-vom-applikationsserver.crt
SSLProxyCipherSuite SSL HIGH:!aNULL:!ARIA:!CAMELLIA:!DHE:!AESCCM:!RSA:!SHA:!AES128+SHA256:!AES256+SHA384
SSLProxyCipherSuite TLSv1.3 TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256

ProxyPass /webapp1 https://applikationsserver.local.net/webapp1
ProxyPassReverse /webapp1 https://applikationsserver.local.net/webapp1

# Beispiel in Location
Redirect permanent /webapp2 /webapp2/
<Location /webapp2/>
    ProxyPass https://applikationsserver.local.net/webapp2/
    ProxyPassReverse https://applikationsserver.local.net/webapp2/
</Location>

# Konfiguration für HTTP-Zugriff zum Anwendungsserver hinter dem ReverseProxy
# ===========================================================================

# Hinweis: Der Zugriff vom ReverseProxy zum Applikationsserverist dadurch unverschluesselt.

ProxyPass /webapp3 http://applikationsserver.local.net/webapp3
ProxyPassReverse /webapp3 http://applikationsserver.local.net/webapp3

# Beispiel in Location
Redirect permanent /webapp4 /webapp4/
<Location /webapp4/>
    ProxyPass http://applikationsserver.local.net/webapp4/
    ProxyPassReverse http://applikationsserver.local.net/webapp4/
</Location>

# Konfiguration für AJP-Zugriff zum Anwendungsserver hinter dem ReverseProxy
# ==========================================================================

# ggf. Request Header ajp vergroessern (default 8192 Byte)
# Die Headergroesse muss evtl. auch am AJP-Connector vom Applikationsserver gesetzt werden.
ProxyIOBufferSize 65536

# Bei Verwendung von mod_ajp muss kein ProxyPassReverse Eintrag gesetzt werden.
# Dies wird teilweise in der Literatur zum Thema teilweise falsch dargestellt.

ProxyPass /webapp5 ajp://applikationsserver.local.net:8009/webapp5


Redirect permanent /webapp6 /webapp6/
<Location /webapp6/>
    ProxyPass ajp://applikationsserver.local.net:8009/webapp6/
</Location>

</VirtualHost>
```