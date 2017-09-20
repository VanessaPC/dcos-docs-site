---
layout: layout.pug
title: Adding external users
menuWeight: 2
excerpt: >
  Once you have configured a directory
  service or an identity provider, you
  should add the users to DC/OS so that
  you can assign them permissions.
featureMaturity:
enterprise: true
navigationTitle:  Adding external users
---


# About adding external users

Once you have configured a directory service or an identity provider, you should add the users to DC/OS so that you can assign them permissions.

# Prerequisites

Before you can add any external users, you must give DC/OS the information it needs to connect to the directory service or identity provider.

* Visit the [LDAP topic](/docs/1.8/administration/id-and-access-mgt/ldap/) for more information on how to set up an external LDAP directory.

* Visit the [SSO topic](/docs/1.8/administration/id-and-access-mgt/sso/) for more information on how to set up an OpenID Connect or SAML provider.

# Requirements

Please refer to the [introduction of this section](/docs/1.8/administration/id-and-access-mgt/users-groups/) for the user name character requirements.

# Adding external users via logon attempt

By default, users have no DC/OS permissions. Any attempts to access DC/OS without permissions will fail. However, if you have successfully configured an LDAP directory or an identity provider and the user provides valid credentials, the logon attempt will cause the user's account to be added to DC/OS.

**Requirement**: The user's name and password must be correct.

Because you will need the user account in DC/OS before you can add any permissions, you may find it easiest to ask each of the users to try to log on to DC/OS. Though their attempts will fail, this will serve to populate DC/OS with their accounts.

# Importing external LDAP users individually from the web interface

If you don't want to ask your users to try to log on, you can import their accounts instead. All you need to know is their user name.

To import an external user:

1. In the **System** -> **Organization** -> **Users** tab, click **Add User**.

2. Select **Import LDAP User**.

3. Type the user's user name or ID in the **User Name** box.

4. Click **Add**.

5. When you have finished adding all of your users, click the **Close** button.


# Importing groups of LDAP users

## About importing LDAP groups

You can import existing LDAP user groups into DC/OS. Importing LDAP groups is a one-time operation: DC/OS does not maintain any connection to the LDAP group after importing.

**Requirement:** Group entries in the LDAP directory must list their members with the `member` attribute.

Group size is limited to 100 users. To increase this limit, contact Mesosphere customer support. If the user name matches an existing user, it is not reimported. You can check the logs to determine if this has occurred.

## Configuring LDAP group import

1. Open the **System** -> **Organization** -> **External Directory** tab.

2. Click **Edit**.

3. Click **Group Import**.

4. Provide the DN for the subset of the directory tree that should be searched in the **Group Search Base** field. Example: `(cn=Users,dc=mesosphere,dc=com)`.

5. Provide a template to be used to translate a group name to a valid LDAP search filter in the **Group Search Filter Template** field. The string must contain `%(groupname)`. Example: `((&(objectclass=group)(sAMAccountName=%(groupname)s)))`.

6. When completed, your dialog should look something like the following.

   ![LDAP Group Import Configuration](../img/ldap-group-import.png)

7. Click **Save Configuration**.

## Importing LDAP groups using the web interface

1. In the **System** -> **Organization** -> **Groups** tab, click **Add Group**.

1. From the drop down menu, select **Import LDAP Group**.

1. Type the LDAP group name in the **Name** box. The group name must not match the name of an existing group.

1. Click **Add Group**. This creates a user group in DC/OS with the same name as the LDAP group and imports all of the users in the LDAP group into DC/OS.

1. When you have finished adding all of your groups, click the **Close** button.


## Importing LDAP groups using the API

You can import a group of LDAP users by using the `/ldap/importuser` [API](/docs/1.8/administration/id-and-access-mgt/iam-api/) endpoint.

**Prerequisites:**

-  The `group-search` configuration key must be set, as discussed in [Configuring LDAP group import](#Configuring-LDAP-group-import).
-  The existing group entries must list their members by using the `member` attribute.

-  If your [security mode](/docs/1.8/administration/installing/ent/custom/configuration-parameters/#security) is `permissive` or `strict`, you must follow the steps in [Obtaining the root certificate of your DC/OS CA](/docs/1.8/administration/tls-ssl/get-cert/) before issuing the curl commands in this section. If your [security mode](/docs/1.8/administration/installing/ent/custom/configuration-parameters/#security) is `disabled`, you must delete `--cacert dcos-ca.crt` from the commands before issuing them.

In this example a group named `johngroup` is imported.

1.  Log into the CLI to ensure that you can reference the cluster URL as shown in the following code samples.

1.  Initiate import with this command:

    ```bash
    curl -i -X POST --cacert dcos-ca.crt -H "Authorization: token=$(dcos config show core.dcos_acs_token)" --data '{"groupname": "johngroup"}' --header "Content-Type: application/json" $(dcos config show core.dcos_url)/acs/api/v1/ldap/importgroup
    ```

1.  Confirm that `johngroup` is added:

    ```bash
    curl --cacert dcos-ca.crt -H "Authorization: token=$(dcos config show core.dcos_acs_token)" $(dcos config show core.dcos_url)/acs/api/v1/groups/johngroup
    ```

    ```bash
    curl --cacert dcos-ca.crt -H "Authorization: token=$(dcos config show core.dcos_acs_token)" $(dcos config show core.dcos_url)/acs/api/v1/groups/johngroup/users
    ```