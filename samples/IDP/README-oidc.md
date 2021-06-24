#  Delegate Authentication to your OIDC identity provider

## Prerequisites
Workflow Process Service is up and running. 

## Register Workflow Process Service as a client of the identity provider
Register Workflow Process Service as a client of your OIDC identity provider.

As a result of this registration, you get a `clientId` and a `clientSecret`  that Workflow Process Service must use to authenticate against the OIDC provider.

## Obtain certificates of the identity provider
Obtain a copy of the certificates of the identity provider, e.g. `idp-cacert.crt`, and move them to the `./config/cert-trusted` directory.


## Configure the Open ID Connect Client 
Edit `./config/liberty/liberty-custom.xml`.

Configure the OIDC client by adding a suitably customized version of the following example to the server configuration:

```
<server>
    <featureManager>
        <feature>openidConnectClient-1.0</feature>
        <feature>ssl-1.0</feature>
    </featureManager>
	
    <keyStore id="oidcKeyStore" location="/shared/custom/tls/trusts.p12" password="${ssl_keystore_password}"/>
	
    <openidConnectClient id="UNIQUE_IDENTIFIER"
        clientId="**********" clientSecret="**********"
        authorizationEndpointUrl="https://server.example.com/oidc/endpoint/default/authorize"
        tokenEndpointUrl="https://server.example.com/oidc/endpoint/default/token"
        validationEndpointUrl="https://server.example.com/oidc/endpoint/default/introspect"
        inboundPropagation="supported"
        signatureAlgorithm="RS256"
        jwkEndpointUrl="https://server.example.com/oidc/endpoint/default/jwks"
        issuerIdentifier="https://server.example.com/oidc/endpoint/default">
        <authFilter>
            <requestUrl matchType="contains" urlPattern="/oidc/endpoint/ums/authorize"></requestUrl>
            <requestHeader id="allowBasicAuth" matchType="notcontain" name="Authorization" value="Basic" />
        </authFilter>
    </openidConnectClient>
</server>
```

Note: 
* Replace the openidConnectClient id with the `UNIQUE-IDENTIFIER` that you specified when your registered the client with your identity provider.
* Replace the `clientId` and `clientSecret` with the values that your received when your registered the client with your identity provider.
* The `authFilter` configuration ensures that only request uris are redirected to the identity provider that
  * contain `/oidc/endpoint/ums/authorize` and
  * do not contain `Basic` in the header `Authorization`

Save the configuration file.

## Edit docker-compose.yaml
Edit `docker-compose.yml` and uncomment the following line to mount `liberty-custom.xml` to `/config/configDropins/overrides/`:
```
      # Custom Liberty configuration dropin (like custom logging, custom user registry). You can mount any file
      # containing Liberty configuration dropins in the /config/configDropins/overrides folder
      # - ./config/liberty/logging.xml:/config/configDropins/overrides/logging.xml
      - ./config/liberty/liberty-custom.xml:/config/configDropins/overrides/liberty-custom.xml
	
```

Uncomment the following line to mount `idp-cacert.crt` to `/shared/custom/cert-trusted/`:
```
      # If IDP is used securely using SSL, you must mount the root certificate which was used to
      # sign the IDP server certificate so that the client can verify that the server's leaf certificate was
      # signed by its trusted root certificate.
      - ./config/cert-trusted/idp-cacert.crt:/shared/custom/cert-trusted/idp-cacert.crt
```

To mount these files, dba-user (id=50001) must have access to these files:
```
chown 50001:0 ./config/liberty/liberty-custom.xml
chmod 400 ./config/liberty/liberty-custom.xml
```

```
chown 50001:0 ./config/cert-trusted/idp-cacert.crt
chmod 400 ./config/cert-trusted/idp-cacert.crt
```

Restart the container.

## Login to Workflow Process Service
Navigate to a Workflow Process Service endpoint e.g: `https://your-host:your-port/Workplace`. You are redirected to the login page of your identity provider.
If the authentication is successful, your will be redirected to the intended endpoint `https://your-host:your-port/Workplace`.

## References
Configuring an OpenID Connect Client in Liberty: 
https://www.ibm.com/docs/en/was-liberty/base?topic=connect-configuring-openid-client-in-liberty