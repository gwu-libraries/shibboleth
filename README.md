shibboleth
==========

The following instructions describe how to configure a Shibboleth Service Provider on an Ubuntu 12.04 LTS server with The George Washington Universtiy as an Identity Provider.  YMMV on other flavors of linux & other Identity Providers.

* Install the required Apache2 module

	% sudo apt-get update
	% sudo apt-get install libapache2-mod-shib2
  
This will create a folder located in /etc/shibboleth that contains the necessary configuration files for Shibboleth.

* Modify the following in the default shibboleth2.xml file

	% sudo vi /etc/shibboleth/shibboleth2.xml

	<ApplicationDefaults entityID="https://sp.example.org/shibboleth"
          REMOTE_USER="eppn persistent-id targeted-id">

  
  	<SSO entityID="https://singlesignon.gwu.edu/idp/shibboleth"
       	  discoveryProtocol="SAMLDS" discoveryURL="https://ds.example.org/DS/WAYF">
          SAML2 SAML1
  	</SSO>
  
  	<MetadataProvider type="XML"
          uri="http://wayf.incommonfederation.org/InCommon/InCommon-metadata.xml"
	  backingFilePath="InCommon-metadata.xml" reloadInterval="7200">
      		<MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
		<MetadataFilter type="Signature" certificate="incommon.pem"/>
  	</MetadataProvider>

* Download the InCommon security keys

	% sudo wget -O /etc/shibboleth/incommon.pem https://wayf.incommonfederation.org/bridge/certs/incommon.pem

* Restart the Shibboleth service daemon

	% sudo service shibd restart
	
* Setup SSL for Apache2

	% sudo a2enmod ssl
