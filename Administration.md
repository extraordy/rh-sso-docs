# Red Hat Single Sign-On Administration

Red Hat Single Sign-On is a solution derived from the upstream project **Keycloak**
whose purpose is to provide a single sign-on interface for web applications and
RESTful web services.  
The goal is to remove the developer's burden of handling authentication in their
application by relying on an external customizable solution.

RH-SSO provides support for **SAML**, **OpenID Connect**, **oAuth2** identity
providers, social logins (Facebook, GitHub, Google, etc), **two-factor**
authentication, user federation to sync with external ActiveDirectory, LDAP and Kerberos.

RH-SSO is and **identity provider (IDP)** to clients that authenticate to a
**Realm**. A Realm can be defined as a collection of users, roles, groups and
credentials.  
**Federation** with extrenal identity providers (ie Facebook) is possible.
Another very common scenario is the federation with LDAP or AD services for
user credentials storage. RH-SSO can point to these services to validate
provided credentials.

### First login
When executed for the first time an administrative user in the **Master** Realm
must be created. To do this, point to URL http://localhost:8080/auth from the
node running the SSO instance. This redirection is available **only** when
pointing to localhost.

![Welcome page](images/initial-welcome-page.png)

If the localhost URL is not available the `.../bin/add-user-Keycloak.sh` script
in the local installation path can be used to achieve the same result.

When running in standalone mode the script must be executed passing the **realm**
name, username and password:

```
$ .../bin/add-user-Keycloak.sh -r master -u <username> -p <password>
```

The realm name must be **master**.

When running in domain mode one of the server instances must be pointed using the
`--sc` flag:

```
$ .../bin/add-user-Keycloak.sh --sc domain/servers/srv-one/configuration \
  -r master -u <username> -p <password>
```

The newly created user can be tested for the first login at the address
http://hostname:8080/auth. If the connection is done to http://localhost:8080/auth
this time no redirection will happen and the main login page will appear:

![Login page](images/login-page.png)

After authenticating with the new credentials the main console page will be
displayed.

![Admin console](images/admin-console.png)

### Realm Management
The **Master** realm settings are displayed, pointing to the **General** tab.
The Master realm is the parent of all realms and is the defaut pre-defined realm.
Admins in this realm can view and manage all other realms created in SSO.

> **_NOTE_**: It is recommended to not use the Master realm in production and
> to reserve it only for superusers to manage the other realms.

For every realm the following tabs are displayed:

- **General**, to configure general settings like the realm name and display name.
- **Login**, to configure login behaviors like email verification of SSL/TLS mode.
- **Keys**, to manage keypairs in the realm.
- **Email**, to configure mail server settings.
- **Themes**, to configure custom themes.
- **Cache**, to manage Realm, User and Keys cache eviction.
- **Tokens**, to manage token lifecycle and SSO sessions.
- **Client Registration**, to manage initial access tokens and client registration
  policies.
- **Security Defenses**, to manage header security options and brute-force
  detection and countermeasures.

#### Realm Creation
To add a new realm click on the drop down menu on the left side: this will show
all the created realms and the button `Add Realm`.  

![Add realm](images/add-realm-menu.png)

By clicking on it the **Add Realm** page is displayed . A new realm can be created
by importing a JSON file of simply by filling the `Name` field and clicking `Create`.

![Create realm](images/create-realm.png)

This action will create a new empty realm ready for customizations.

#### Realm Keys
When a new Realm is created a new keypair and a self-signed certificate are
issued for the realm. They will be used to create new signatures: all tokens and
cookies are signed with the keypair.   

Along with the active keypair RH-SSO can hold a set of rotated passive keypairs to verify previous signatures. This lets the administrators to rotate the keypairs with no downtime.  

Keypairs can be seen in the **Keys** tab.

![Realm keys](images/realm-keys.png)

The **Active** and **Passive** sub-tabs show the current active key and the
passivated keys. It's recummended to regurarly rotate the keys.
To add a new keypair click on the **Providers** sub-tab and then select from
the **Add keystore...** menu the correct provider. Select **rsa-generated** to
create a new RSA keypair. Configure the key priority to the desired value.
Active keypair can coexist in a Realm and the keypair with the highest priority
will be used.

![Add RSA provider](images/add-rsa-provider.png)

Existing keypairs can be passivated or disabled by configuring their provider.

#### Ream Themes
After creating a Realm themes can be customized.
A theme consists of:

- HTML templates
- Images
- Message bundles
- Stylesheets
- Scripts
- Theme properties

Themes are installed in the `themes` directory in the RH-SSO installation path.
By default three themes are installed, **base**, **keycloak** and **rh-sso**.
These defaault themes shouldn't be modified. Instead, new themes should be
added to extend the build-int themes.

The themes section in the Realm Settings offer four different theme customization
choices:

- Login Theme
- Account Theme
- Admin Console Theme
- Email Theme

The **Internationalization** switch enables users to choose their language in
every theme.

![Themes](images/themes-tab.png)

### User Management
Users can be managed by clicking in the left menu `Users` item.
By clicking on the `Add User` button a new user can be created. The only
mandatory field is the **username**. Users cna be created **Enabled** by
default or **Disabled**. Administrators can force the users to verify their
email address or not with the `Email verified` boolean.
The `Requires User Actions` fields is a set of actions done when the user logs
in:

- **Verify email** sends an email to the user to verify his/her address.
- **Update profile** forces user to update his/her profile upon first login.
- **Update password** forces user to update the password.
- **Configure OTP** forces OTP configuration after first login.

Default required actions can be enabled/dsabled by clicking the **Authentication**
section in the left menu and selecting the **Required Actions** tab.

![Add user](images/add-user.png)

Newly created users will appear in the main Users tab. For every user the Realm
administrator can choose the following actions:

- `Edit` to modify the user settings.
- `Impersonate` to become the user for testing purposes.
- `Delete` to remove the user.

![User actions](images/delete-user.png)

#### User Credentials
Realm admins can set users passwords in the **Credentials** tab in the user
settings. The password can be set as **Temporary** to force the change upon next
user login.

![User credentials](images/user-credentials.png)
