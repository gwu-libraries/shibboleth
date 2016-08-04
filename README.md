shibboleth
==========

The following instructions describe how to configure a Shibboleth Service Provider on an Ubuntu 12.04 LTS server with The George Washington Universtiy as an Identity Provider.  YMMV on other flavors of linux & other Identity Providers.

* Install the required Apache2 module

```
	 % sudo apt-get update
	 % sudo apt-get install libapache2-mod-shib2
```

This will create a folder located in /etc/shibboleth that contains the necessary configuration files for Shibboleth.

Configure the Shibboleth settings for your Service Provider:

* Modify the following in the default shibboleth2.xml file
```
	 % sudo vi /etc/shibboleth/shibboleth2.xml
```

* Modify the entityID line of the config file to reflect the name of your server:

```
	 <ApplicationDefaults entityID="https://sp.example.org/shibboleth"
          REMOTE_USER="eppn persistent-id targeted-id">
```

* Replace the SSO entityID section with the following:

```
	 <SSO entityID="https://singlesignon.gwu.edu/idp/shibboleth"
       	  discoveryProtocol="SAMLDS" discoveryURL="https://ds.example.org/DS/WAYF">
          SAML2 SAML1
	 </SSO>
```

* Reaplce the MetadataProvider section with the following:

```
	 <MetadataProvider type="XML"
	 uri="http://wayf.incommonfederation.org/InCommon/InCommon-metadata.xml" backingFilePath="InCommon-metadata.xml" reloadInterval="7200">
      		<MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
	 	<MetadataFilter type="Signature" certificate="incommon.pem"/>
	 </MetadataProvider>
```

* Download the InCommon security keys

```
	 % sudo wget -O /etc/shibboleth/incommon.pem https://wayf.incommonfederation.org/bridge/certs/incommon.pem
```

* Copy and customize the localLogout.html file for your application:

```
cp localLogout.html.template localLogout.html
```

Modify the page title in the head section (around line 5), and the application names and urls in lines 63, 85, and 87 as approriate.

* Enable the shib2 module for Apache

```
	 % sudo a2enmod shib2
```

* Generate an  x.509 certificate for Shibboleth to use:

```
	 % sudo shib-keygen
```

* Restart the Shibboleth service daemon

```
	 % sudo service shibd restart
```

Shibboleth requires SSL for its transactions.  Setup SSL:
	
* Setup SSL for Apache2

```
	 % sudo a2enmod ssl
```

Generate server keys and a certificate signing request:

* Create server key

```
	 % cd ~
	 % openssl genrsa -des3 -out hostname.key 2048
```

You will be asked to set a pass phrase for the key, don't lose this.

```	 
	 % openssl rsa -in hostname.key -out hostname.key.insecure
	 % mv hostname.key hostname.key.secure
	 % mv hostname.key.insecure hostname.key
```

* Create a certificate signing request

```
	 % openssl req -new -key hostname.key -out hostname.csr
```

Download the hostname.csr file and attach it to an email to ithelp@gwu.edu.  In the body of the message request an InCommon signed certificate and specify that you are using Apache2 as your webserver.  The Division of IT will return to you an email that you can download a hostname.cert file for your server.

* Upload the hostname.cert file to your server.

* Install the hostname.cert file on your server

```
	 % sudo mv hostname.cert /etc/ssl/certs
```

* Install the hostname.key file your generated earlier:

```
	 % sudo mv hostname.key /etc/ssl/private
```

Create an Apache2 virtual host file for SSL ie: default-ssl

* Enable the virtual host file for SSL

```
	 % sudo a2ensite default-ssl
```

Add the Shibboleth configurations to your SSL virtual host file:

```
	 % sudo vi /etc/apache2/sites-available/default-ssl
```

```
	<Location "/Shibboleth.sso">
	 SetHandler shib-handler
	</Location>
      	
	<Location /secure>
	 # This is an example Location directive that redirects apache over to the IdP.
	 AuthType shibboleth
	 ShibRequestSetting requireSession 1
	 require valid-user
	</Location>
        
	SSLEngine On
	SSLOptions +StrictRequire
	SSLProtocol ALL -SSLv2 -SSLv3
	SSLHonorCipherOrder On
	SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
	SSLCertificateFile /etc/ssl/certs/hostname.cert
	SSLCertificateKeyFile /etc/ssl/private/hostname.key
	SSLCertificateChainFile /etc/ssl/intermediate/incommon-ssl.ca-bundle
```

Restart Apache2

```
	 % sudo service apache2 restart
```

Download your Shibboleth Service Provider metadata file	

* Navigate to https://sp.example.org/Shibboleth.sso/Metadata (and substituting your server's name for sp.example.org)

Download the Metadata.xml file and rename it to hostname-metadata.xml. Attach the file to an email to ithelp@gwu.edu.  In the body of the message request that your service provider be registered with the GWU Shibboleth Identify provider.  Make note that the metadata.xml file is attached to the email.

Once your Service provider has been registered you should be able to navigate to your server and test Shibboleth with the GWU Identity Provider.

* Navigate to https://sp.example.org/secure
