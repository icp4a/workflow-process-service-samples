# Workflow Process Service Samples

Workflow Process Service can be run in the Workflow Process Service Authoring environment or the Workflow Process Service Server environment. The authoring environment comes with IBM® Business Automation Studio and allows you to create, maintain, and edit Business Automation Workflow. Both environments run Business Automation Workflow and have IBM Workplace as the front-end user interface. Each of these environments has the same prerequisites but they have unique Docker images.

# Overview

The production folder gives an example of using docker compose to run Workflow Process Service. Before calling docker-compose up, you must configure [ docker-compose.yml ](./production/docker-compose.yml) carefully.

The samples folder describes LDAP/IDP integration scenarios to define the users and groups information for authentication in Workflow Process Service environment.
- [ Connect Workflow Process Service to Active Directory with SSL enabled ](./samples/LDAP/README-Connect-to-AD.md)
- [ Connect Workflow Process Service to IBM Directory Server with SSL enabled ](./samples/LDAP/README-Connect-to-IDS-with-SSL.md)
- [ Connect Workflow Process Service to IBM Directory Server with no SSL enabled ](./samples/LDAP/README-Connect-to-IDS-no-SSL.md)
- [ Delegate Authentication to your OIDC identity provider ](./samples/IDP/README-oidc.md)
- [ Delegate Authentication to your SAML identity provider ](./samples/IDP/README-saml.md)
