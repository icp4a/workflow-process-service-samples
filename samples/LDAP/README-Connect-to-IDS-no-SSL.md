#  Connect Workflow Process Service to an IBM Directory Server with no SSL enabled

## Prerequisites
Workflow Process Service is up and running. 

## Provide LDAP Configuration
Create `./config/liberty-custom.xml`.

Adapt the following example to suit your LDAP setup and add it to the server configuration in `./config/liberty-custom.xml`.

```
<server>
    <featureManager>
        <feature>ldapRegistry-3.0</feature>
    </featureManager>
    <ldapRegistry id="LdapConfig" realm="defaultRealm"
              host="ldapserver.mycity.mycompany.com"
              baseDN="o=mycompany,c=us"
              port="389"
              recursiveSearch="false"
              sslEnabled="false"
              ldapType="IBM Tivoli Directory Server" >
			  
        <loginProperty name="uid" />
        <loginProperty name="mail" />
	  
        <ldapEntityType id="entityTypePA" name="PersonAccount">
            <objectClass>ePerson</objectClass>
            <searchBase>ou=Users,o=mycompany,c=us</searchBase>
        </ldapEntityType>

        <ldapEntityType id="entityTypeGroup" name="Group">
            <objectClass>groupOfUniqueNames</objectClass>
            <searchBase>ou=Group,o=mycompany,c=us</searchBase>
            <searchFilter>cn=BPM*</searchFilter>
        </ldapEntityType>

        <groupProperties>
            <memberAttribute name="allMembers" objectClass="groupOfUniqueNames" scope="all"/>
            <membershipAttribute name="allGroups" scope="all" />
        </groupProperties>

        <attributeConfiguration>
            <attribute name="givenName" propertyName="givenName" syntax="String" entityType="PersonAccount" />
            <attribute name="givenName" propertyName="cn" syntax="String" entityType="PersonAccount" />
            <attribute name="sn" propertyName="familyName" syntax="String" entityType="PersonAccount" />
            <attribute name="callupName" propertyName="displayName" syntax="String" entityType="PersonAccount" />
            <attribute name="mail" propertyName="mail" syntax="String" entityType="PersonAccount" />
        </attributeConfiguration>
    </ldapRegistry>
	
    <federatedRepository id="vmm" maxSearchResults="100100">
        <extendedProperty dataType="String" name="externalId" entityType="Group"></extendedProperty>
        <extendedProperty dataType="String" name="externalId" entityType="PersonAccount"></extendedProperty>
    </federatedRepository>
</server>
```

Note: 
* A custom LDAP search expression can be used when searching for entity types. This expression can be defined by specifying a search filter.
In this example, the search filter `cn=BPM` is used which will return all entities where the field CN starts with `BPM`. 
* Define a meaningful value for the `maxSearchResults`. The `maxSearchResults` should be larger than the number of groups returned when the filter is applied.

Save the configuration file.

## Edit docker-compose.yaml
Edit docker-compose.yml and uncomment the following line to mount `liberty-custom.xml` to `/config/configDropins/overrides/`
```
      # Custom Liberty configuration dropin (like custom logging, custom user registry). You can mount any file
      # containing Liberty configuration dropins in the /config/configDropins/overrides folder
      # - ./config/liberty/logging.xml:/config/configDropins/overrides/logging.xml
      - ./config/liberty/liberty-custom.xml:/config/configDropins/overrides/liberty-custom.xml
	
```

To mount this file, dba-user (id=50001) must have access to it:
```
chown 50001:0 ./config/liberty/liberty-custom.xml
chmod 400 ./config/liberty/liberty-custom.xml
```

Restart the container.

## Login with an IBM Directory Server user

Verify that you can login using an LDAP user ID. Navigate to a Workflow Process Service endpoint, such as https://your-host:your-port/Workplace.
You are redirected to the login page, where you can enter a valid LDAP user name and password.
If the authentication is successful, you will be redirected to the intended endpoint https://your-host:your-port/Workplace.

## References
Configuring LDAP user registries in Liberty: 
https://www.ibm.com/docs/en/was-liberty/base?topic=liberty-configuring-ldap-user-registries-in