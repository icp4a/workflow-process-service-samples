#  Delegate Authentication to your SAML identity provider

## Prerequisites
Workflow Process Service is up and running. 

## Obtain the metadata and certificates from your identity provider

From your identity provider download the metadata to a file, e.g. `idpMetadata.xml`.
Copy the `idpMetadata.xml` to `./config/liberty/`.

Obtain a copy of the certificates of the identity provider, e.g. `idp-cert.pem`, and move them to the `./config/cert-trusted` directory.

## Configure SAML
Edit `./config/liberty/liberty-custom.xml`.

Add the following configuration to the server configuration:

```
<server>
  <featureManager>
    <feature>samlWeb-2.0</feature>
  </featureManager>
  <samlWebSso20 id="defaultSP" httpsRequired="true">
    <authFilter>
      <userAgent id="disable" matchType="equals" agent="samldefault" />
    </authFilter>
  </samlWebSso20>
  <samlWebSso20 id="umsSP"
    httpsRequired="true"
    allowCustomCacheKey="false"
    spLogout="true"
    disableLtpaCookie="false"
    mapToUserRegistry="User"
    idpMetadata="/config/configDropins/overrides/idpMetadata.xml" >
    <authFilter>
      <requestHeader id="allowBasicAuth" matchType="notcontain" name="Authorization" value="Basic" />
      <requestUrl id="authorizeEndpoint" urlPattern="/oidc/endpoint/ums/authorize" matchType="contains"/>
    </authFilter>
  </samlWebSso20>
 </server>
```

## Edit docker-compose.yaml
Edit `docker-compose.yml` and uncomment the following line to mount `liberty-custom.xml` to `/config/configDropins/overrides/`:
```
      # Custom Liberty configuration dropin (like custom logging, custom user registry). You can mount any file
      # containing Liberty configuration dropins in the /config/configDropins/overrides folder
      # - ./config/liberty/logging.xml:/config/configDropins/overrides/logging.xml
      - ./config/liberty/liberty-custom.xml:/config/configDropins/overrides/liberty-custom.xml
```

Uncomment the following line to mount `idpMetadata.xml` to `/config/configDropins/overrides/`:
```
      # SAML IDP Metadata configuration
      - ./config/liberty/idpMetadata.xml:/config/configDropins/overrides/idpMetadata.xml	
```

Uncomment the following line to mount `idp-cert.pem` to `/shared/custom/cert-trusted`:
```
      # If IDP is used securely using SSL, you must mount the root certificate which was used to
      # sign the IDP server certificate so that the client can verify that the server's leaf certificate was
      # signed by its trusted root certificate.
      - ./config/cert-trusted/idp-cert.pem:/shared/custom/cert-trusted/idp-cert.pem
```

To mount these files, dba-user (id=50001) must have access to these files:
```
chown 50001:0 ./config/liberty/liberty-custom.xml
chmod 400 ./config/liberty/liberty-custom.xml
```
```
chown 50001:0 ./config/liberty/idpMetadata.xml
chmod 400 ./config/liberty/idpMetadata.xml
```
```
chown 50001:0 ./config/cert-trusted/idp-cert.pem
chmod 400 ./config/cert-trusted/idp-cert.pem
```

Restart the container.

Note: SAML will not yet work at this point, you need to register Workflow Process Service with your identity provider.

## Download spMetadata.xml and register your Workflow Process Service 

As you have enabled SAML in Workflow Process Service in the previous step, download the `spMetadata.xml` file from https://yourhost:yourport/ibm/saml20/umsSP/samlmetadata.
Register your Workflow Process Service as a client of your SAML identity provider by providing the spMetadata.xml.

## Login to Workflow Process Service

Navigate to a Workflow Process Service endpoint e.g: `https://your-host:your-port/Workplace`.  You are redirected to the login page of your identity provider.
If the authentication is successful, your will be redirected to the intended endpoint `https://your-host:your-port/Workplace`.

## References
Configuring SAML Web Browser SSO in Liberty
https://www.ibm.com/docs/en/was-liberty/base?topic=liberty-configuring-saml-web-browser-sso-in