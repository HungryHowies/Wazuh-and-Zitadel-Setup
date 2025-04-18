# Wazuh SSO setup

## Overview

The following documentation explains the configurations needed for Wazuh Single sign-on (SSO) and the connection to Zitadel instance. 
Wazuh node must be in Production mode, meaning you have created the certificate for "node/s, admin and CA" and ensure HTTPS is working correct. 
Take note this is a basic configuration setup to start SSO with wazuh using Zitadel.

To use SAML for authentication, configurations are needed in the **authc** section of this file 

```
vi /etc/wazuh-indexer/opensearch-security/config.yml
```

Setup authentication_backend to noop. Place all SAML-specific configuration options in config.yml file, under the section *saml_auth_domain:*. Ensure the order number is correct. In the example below the saml_auth_domain ORDER is set to 1 and basic_internal_auth_domain is set to "0". The  basic_internal_auth_domain challenge is set from true to false.


NOTE: The Security plugin can read IdP metadata either from a URL or a file. In this example Im using URL.

### Edit config.conf file.

```
vi /etc/wazuh-indexer/opensearch-security/config.yml
```
Copy and paste this blank configuration under the *authc* section. In this documentation I placed it under basic_internal_auth_domain:.

When completed it should look like this, ensure all indents are correct and I have shown http_enabled: is set to true. 

```
saml_auth_domain:
        http_enabled: true
        transport_enabled: false
        order: 1
        http_authenticator:
          type: saml
          challenge: false
          config:
            idp:
              metadata_url: 
              entity_id: 
            sp:
              entity_id: 
            kibana_url: 
            subject_key: 
            roles_key: Role
            exchange_key:             
        authentication_backend:
          type: noop
```
### Configure section "saml_auth_domain"

Get the exchange_key needed for Wazuh , you need to create a service_user in Zitadel.

Login to Zitadel Dashboard then navigate to Organization --> Users.

Under the Users section click "Service Users"

![image](https://github.com/HungryHowies/Zitadel-with-Opensearch-SSO/assets/22652276/a37115d0-784c-429d-a0ea-3b522fc412a2)


 When the Service User is completed, on the left pane click "Personal Access Tokens" and click "New".
 
![image](https://github.com/HungryHowies/Wazuh-and-Zitadel-Setup/assets/22652276/eb5929f3-0152-4f22-b046-b791886dfa0d)


Copy the token from Zitadel service_user.

![image](https://github.com/HungryHowies/Zitadel-with-Opensearch-SSO/assets/22652276/afaed1f4-f08e-4357-b695-d8b37e3603ed)

Paste service_user token from Zitadel to the exchange_key section in the config.yml file.

```
vi /etc/wazuh-indexer/opensearch-security/config.yml
```
Results:

```
exchange_key: AwqgAwIBAgICAY4wDQYJKoZIhvcNANjA2NT1UEChC0SOMETHING
```


### Zitadels metadata URL

For the *metadata_url* and *entity_id* section, I used Zitadel metadata URL.

```
https://zitadel-self-hosting.com/saml/v2/metadata
```

Add the following SAML settings in the config.yml file under *authc: saml_auth_domain*

```
subject_key: Email
challenge: true
metadata_url: https://zitadel.self-hosting.com/saml/v2/metadata
entity_id: https://zitadel.self-hosting/saml/v2/metadata
```

The completed saml configuration is shown below.

```
  authc:
      saml_auth_domain:
       http_enabled: true
       transport_enabled: true
       order: 1
       http_authenticator:
        type: saml
        challenge: true
        config:
         idp:
          metadata_url: https://zitadel.self-hosting.com/saml/v2/metadata
          entity_id: https://zitadel.self-hosting/saml/v2/metadata
         sp:
          entity_id: https://wazuh.domain.com:5601
         kibana_url: https://wazuh.domain.com:5601
         subject_key: Email
         roles_key: Role
         exchange_key: AwqgAwIBAgICAY4wDQYJKoZIhvcNANjA2NT1UEChC0SOMETHING
       authentication_backend:
          type: noop
  ```



NOTE: I did add a section for LOGOUT as shown below. It is critial that the setting entityID and the entity_ID: in Wazuh config.yml file match exactly.

```
<?xml version="1.0"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"                     
                     entityID="https://wazuh.domain.com">
    <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
        <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
                                Location="https://wazuh.domain.com/_opendistro/_security/saml/logout" />
        <md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</md:NameIDFormat>
        <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
                                     Location="https://wazuh.domain.com/_opendistro/_security/saml/acs" index="0" />
        
    </md:SPSSODescriptor>
</md:EntityDescriptor>
```

Click Continue, then create.


### Service user to Zitadel Project

Give the service_user a role called "Project Owner Viewer Global".

Add the new  "Service_User"  in the Authorizations section for the Wazuh Project. 

![image](https://github.com/HungryHowies/Wazuh-and-Zitadel-Setup/assets/22652276/c5526860-ceab-43b3-ad78-bcc5b64b2cd4)



### Wazuh Add User to Role

Adding user from Zitadel Project.

Login to wazuh with Default Admin credentials. 

Navigate to Security --> Roles.

![image](https://github.com/HungryHowies/Wazuh-and-Zitadel-Setup/assets/22652276/a3aefffd-0f38-4ddc-bc5e-c6ada400e3dd)


 Add the user from Zitadel to a default Role or custom Role in wazuh. 
 
 **Example:** I added some.user from Zitadel to **all_access**. 

 Choose "all_access", then click the Mapped Users tab.

 Button upper right, click "Manage mapping". Add the user "some.user".
 
 
 ![image](https://github.com/HungryHowies/Zitadel-with-Opensearch-SSO/assets/22652276/e4451297-0316-4a67-bf58-47a750463041)

WEB UI should look like this. 

 ![image](https://github.com/HungryHowies/Wazuh-and-Zitadel-Setup/assets/22652276/71f13c80-190c-4f6e-9b12-5a475c11d70f)


 You can either use a internal user (admin) to login or SSO button that would login a user from Zitadel.


 ### Wazuh/Opensearch Logging off with 404

 When logging off,  I recieved a 404 error.

***{"statusCode":404,"error":"Not Found","message":"Not Found"}***
 
Found the solution   [Here](https://forum.opensearch.org/t/saml-issue-on-logout/5617/16?u=gsmitt)

What I did was edit the following file. Line (326,15). If its not there execute a searchin the file for **redirectUrl**

```
vi /usr/share/wazuh-dashboard/plugins/securityDashboards/server/auth/types/saml/routes.js
```
Commented out this line.

```
//  const redirectUrl = authInfo.sso_logout_url || this.coreSetup.http.basePath.serverBasePath || '/';
```

Added this line.

```
const redirectUrl = `${this.coreSetup.http.basePath.serverBasePath}/app/home`;
```

Results:

![image](https://github.com/HungryHowies/Zitadel-with-Opensearch-SSO/assets/22652276/fc0f0851-5ac2-4010-988b-4560ce2c210d)

Restart Wazuh Dashboard service
```
systemctl restart wazuh-dashboard
```
