# Red Hat Single Sign-On Admin CLI

This documents is a quick overview of the Admin CLI provided with Red Hat
Single Sign-On.

The Admin CLI comes packages with the RH-SSO installation and is available for
both Windows and Linux platforms. The purpose of the Admin CLI is to manage all
the resources from command line.
Data is represented in JSON format.

### Invocation
To invoke the Admin CLI on Linux machines:

```
$ /path/to/keycloak/bin/kcadm.sh <COMMAND> <OPTIONS>
```

Where `/path/to/keycloak` is the imaginary path where the RH-SSO (or Keycloak)
installation resides. This path is the `JBOSS_HOME` of the application server.

To invoke the CLI on Windows machines:

```
c:\> kcadm <COMMAND> <OPTIONS>
```

This document will focus on the usage on Linux machines.

Users who are intended to use the CLI must have access read and execute
permissions on the file.
A good practice is to include the path of the SSO bin folder in the `PATH`
variable:

```
export PATH=$PATH:/opt/rh-sso-7.3/bin
```

Another useful customization can be the creation a symlink in a folder already
included in the search path with the shortest name `kcadm`, removing the extension.

```
sudo ln -s /opt/rh-sso-7.3/bin/kcadm.sh /usr/local/bin/kcadm
```

### authentication
The first step necessary to use the Admin CLI is to authenticate to RH-SSO
providing a realm and user credentials:

```
$ kcadm.sh config credentials --server http://localhost:8080/auth --realm master \
  --user admin --password redhat
```

If authentication succeeds the file `~/.keycloak/kcadm.config` will be created.
This file will hold informations about the endpoints, the token and the refresh
token:

```
{
  "serverUrl" : "http://localhost:8080/auth",
  "realm" : "master",
  "endpoints" : {
    "http://localhost:8080/auth" : {
      "master" : {
        "clientId" : "admin-cli",
        "token" : "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJEM3Bzc0MtRnp1VzNsSndXTFViaDI3aDByUGxBVWtZMFRSS0VMZjZ1VVJFIn0.eyJqdGkiOiI2YzUyNThjMS1hNzY1LTRiNmQtOGJmNC02NDY0OTMwMmQ1NmQiLCJleHAiOjE1NTUzNjg1MDgsIm5iZiI6MCwiaWF0IjoxNTU1MzY4NDQ4LCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvbWFzdGVyIiwic3ViIjoiN2Q2NDg4YzAtMWUwNi00NTczLThiZGEtYzcyNjdiODhjMmI4IiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiYWRtaW4tY2xpIiwiYXV0aF90aW1lIjowLCJzZXNzaW9uX3N0YXRlIjoiZWI1ZjkyODYtYzQ4MC00YWFmLTliN2UtNzBmZjgzMzFlMjE5IiwiYWNyIjoiMSIsInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwicHJlZmVycmVkX3VzZXJuYW1lIjoiYWRtaW4ifQ.DTJgCGhcMF5PWoeqmfjv6ZqQolpCZuRIuMmN0UVKxYOD-3zTnLvhglCiqWMypXKyEwAhrb7xvG2qAZTdGob9bJVYMvjoIXUS799i4jTNazHAbJrQT3LGy2WAQ_kE_uP9N4tHHeAjNyJAPUFEquBzPs0MGur28QmL3yvii30hGU_SLZPQh-cdNC_my7blfzy6r0kZijWM2gVyzO8_JI3jLOBurTObnDuoCglzVw_5IdJmrtLEezCUMBe2MXZ47fI4PC0n73jkQr2APFlyjj-5yNmVUEJurNPbdWi3fMgnPD29U1xNdTRDs6ayTXq3axljnZWf-7s3TpGAv6z2YDUPqw",
        "refreshToken" : "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIyODgxMmVmZi0zNDc4LTRiOGQtYWNiMS04NGM5MTFkMGI1MDAifQ.eyJqdGkiOiJkZWYyMGI2Yi04ZTEyLTQ0MTYtYmVmNC0yOThmNDgyOWY4YWQiLCJleHAiOjE1NTUzNzAyNDgsIm5iZiI6MCwiaWF0IjoxNTU1MzY4NDQ4LCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvbWFzdGVyIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL21hc3RlciIsInN1YiI6IjdkNjQ4OGMwLTFlMDYtNDU3My04YmRhLWM3MjY3Yjg4YzJiOCIsInR5cCI6IlJlZnJlc2giLCJhenAiOiJhZG1pbi1jbGkiLCJhdXRoX3RpbWUiOjAsInNlc3Npb25fc3RhdGUiOiJlYjVmOTI4Ni1jNDgwLTRhYWYtOWI3ZS03MGZmODMzMWUyMTkiLCJzY29wZSI6ImVtYWlsIHByb2ZpbGUifQ.s3Taiv9GY-C9sJ1Yr7_WNd81h4pQmb7wAllbTCJXP1w",
        "expiresAt" : 1555368508717,
        "refreshExpiresAt" : 1555370248717
      }
    }
  }
```

After the first login commands can be issued without any further authentication
since the config file will be used by default. To point to a different
configuration file (referred to a different SSO instance, for example), use the
`--config` option followed by the config file path.

### Basic commands
To get a list of the basic commands from the CLI use the `--help` flag:

```
$ kcam.sh --help
Keycloak Admin CLI

Use 'kcadm.sh config credentials' command with username and password to start a session against a specific
server and realm.

For example:

  $ kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin
  Enter password:
  Logging into http://localhost:8080/auth as user admin of realm master

Any configured username can be used for login, but to perform admin operations the user
needs proper roles, otherwise operations will fail.

Usage: kcadm.sh COMMAND [ARGUMENTS]

Global options:
  -x            Print full stack trace when exiting with error
  --help        Print help for specific command
  --config      Path to the config file (~/.keycloak/kcadm.config by default)

Commands:
  config        Set up credentials, and other configuration settings using the config file
  create        Create new resource
  get           Get a resource
  update        Update a resource
  delete        Delete a resource
  get-roles     List roles for a user or a group
  add-roles     Add role to a user or a group
  remove-roles  Remove role from a user or a group
  set-password  Re-set password for a user
  help          This help

Use 'kcadm.sh help <command>' for more information about a given command.
```

### Viewing resources
To get informations about resources use the `get` command. For example, to see
the configuration of the **training** realm use the following command:

```
$ kcadm.sh get realms/training
{
  "id" : "training",
  "realm" : "training",
  "notBefore" : 0,
  "revokeRefreshToken" : false,
  "refreshTokenMaxReuse" : 0,
  "accessTokenLifespan" : 300,
  "accessTokenLifespanForImplicitFlow" : 900,
  "ssoSessionIdleTimeout" : 1800,
  "ssoSessionMaxLifespan" : 36000,
  "ssoSessionIdleTimeoutRememberMe" : 0,
  "ssoSessionMaxLifespanRememberMe" : 0,
  "offlineSessionIdleTimeout" : 2592000,
  "offlineSessionMaxLifespanEnabled" : false,
  "offlineSessionMaxLifespan" : 5184000,
  "accessCodeLifespan" : 60,
  "accessCodeLifespanUserAction" : 300,
  "accessCodeLifespanLogin" : 1800,
  "actionTokenGeneratedByAdminLifespan" : 43200,
  "actionTokenGeneratedByUserLifespan" : 300,
  "enabled" : true,
  "sslRequired" : "external",
  "registrationAllowed" : true,
  "registrationEmailAsUsername" : false,
  "rememberMe" : false,
  "verifyEmail" : false,
  "loginWithEmailAllowed" : true,
  "duplicateEmailsAllowed" : false,
  "resetPasswordAllowed" : false,
  "editUsernameAllowed" : false,
  "bruteForceProtected" : false,
  "permanentLockout" : false,
  "maxFailureWaitSeconds" : 900,
  "minimumQuickLoginWaitSeconds" : 60,
  "waitIncrementSeconds" : 60,
  "quickLoginCheckMilliSeconds" : 1000,
  "maxDeltaTimeSeconds" : 43200,
  "failureFactor" : 30,
  "defaultRoles" : [ "offline_access", "uma_authorization" ],
  "defaultGroups" : [ "/rh-training/curriculum-team" ],
  "requiredCredentials" : [ "password" ],
  "otpPolicyType" : "totp",
  "otpPolicyAlgorithm" : "HmacSHA1",
  "otpPolicyInitialCounter" : 0,
  "otpPolicyDigits" : 6,
  "otpPolicyLookAheadWindow" : 1,
  "otpPolicyPeriod" : 30,
  "otpSupportedApplications" : [ "FreeOTP", "Google Authenticator" ],
  "browserSecurityHeaders" : {
    "contentSecurityPolicyReportOnly" : "",
    "xContentTypeOptions" : "nosniff",
    "xRobotsTag" : "none",
    "xFrameOptions" : "SAMEORIGIN",
    "xXSSProtection" : "1; mode=block",
    "contentSecurityPolicy" : "frame-src 'self'; frame-ancestors 'self'; object-src 'none';",
    "strictTransportSecurity" : "max-age=31536000; includeSubDomains"
  },
  "smtpServer" : { },
  "eventsEnabled" : false,
  "eventsListeners" : [ "jboss-logging" ],
  "enabledEventTypes" : [ ],
  "adminEventsEnabled" : false,
  "adminEventsDetailsEnabled" : false,
  "internationalizationEnabled" : false,
  "supportedLocales" : [ ],
  "browserFlow" : "browser",
  "registrationFlow" : "registration",
  "directGrantFlow" : "direct grant",
  "resetCredentialsFlow" : "reset credentials",
  "clientAuthenticationFlow" : "clients",
  "dockerAuthenticationFlow" : "docker auth",
  "attributes" : {
    "_browser_header.xXSSProtection" : "1; mode=block",
    "_browser_header.xFrameOptions" : "SAMEORIGIN",
    "_browser_header.strictTransportSecurity" : "max-age=31536000; includeSubDomains",
    "permanentLockout" : "false",
    "quickLoginCheckMilliSeconds" : "1000",
    "_browser_header.xRobotsTag" : "none",
    "maxFailureWaitSeconds" : "900",
    "minimumQuickLoginWaitSeconds" : "60",
    "failureFactor" : "30",
    "actionTokenGeneratedByUserLifespan" : "300",
    "maxDeltaTimeSeconds" : "43200",
    "_browser_header.xContentTypeOptions" : "nosniff",
    "offlineSessionMaxLifespan" : "5184000",
    "actionTokenGeneratedByAdminLifespan" : "43200",
    "_browser_header.contentSecurityPolicyReportOnly" : "",
    "bruteForceProtected" : "false",
    "_browser_header.contentSecurityPolicy" : "frame-src 'self'; frame-ancestors 'self'; object-src 'none';",
    "waitIncrementSeconds" : "60",
    "offlineSessionMaxLifespanEnabled" : "false"
  },
  "userManagedAccessAllowed" : false
}
```

This will produce a JSON formatted output.

### Updating resources
The previuosly illustrated JSON output can be redirected to a file for further
offline editing:

```
$ kcadm.sh get realms/training > training.json
```

After editing the output JSON file the updated version can be imported again
using the `update` command:

```
$ kcadm.sh update realms/training -f training.json
```

To update a resource directly from the CLI use the the `-s` option followed by
the updated key/value pair:

```
$ kcadm.sh update realms/training -s verifyEmail=true
```

This will update the **_verifyEmail_** field.

The update command can combine the file import feature and the inline key/value
update:

```
$ kcadm.sh update realms/training -f training.json -s verifyEmail=true
```

This command basically imports the JSON manifest with the resource settings
and overrides one or more key/value pairs on the fly.

### Creating resources
To create a new resource use the `create` command followed by the resource
details. Again, JSON files containing resource definitions can be used in a
manner similar to the `import` functionality in the web interface.

The following example creates a realm named **demo**
```
$ kcadm.sh create realms -s realm=demorealm
```

If a JSON manifest containing the resource informations is available, it can be
used with the `-f` option:

```
$ kcadm.sh create realms -f demo.json
```

Once again, the `-s` and `-f` options can be combined together to override the
contents of the manifest file:
```
$ kcadm.sh create realms -f demo.json -s enabled=true -s  registrationAllowed=true
```

Resources in a realm can be created using the `-r` option to specify the realm
name. The following example creates a user **demouser** with the **enabled=true**
key/value set:
```
$ kcadm.sh create user -r demorealm -s username=demouser -s enabled=true
Created new user with id 'df72ab9b-055b-4f40-aec4-3cba3003c2a8'
```

The same result can by achieved by passing a JSON payload from the standard
input:
```
$ kcadm create realms -f - << EOF
{ "realm": "demorealm", "enabled": true}
EOF
```

This approach can be useful when command are scripted and values are passed as
paramenters in the shell script.

### Deleting resources
To delete a resource use the `delete` command. The following example deletes
the previuosly created **demouser** user in the realm named **demorealm**:

```
$ kcadm.sh delete users/df72ab9b-055b-4f40-aec4-3cba3003c2a8 -r demorealm
```

### Getting user and group roles
The Admin CLI has a dedicate command for listing user's roles in a realm. The
command is `get-roles` and can be issued against a specific user or group.

The following example shows the output of the command by passing the user uid
with the `--uid` flag and the realm name.

```
$ kcadm get-roles --uid d8162cb4-f9a8-4e2e-88c5-16a6c48d5b03 -r demorealm
[ {
  "id" : "58219141-d7ae-4c74-a736-a955a9898e16",
  "name" : "offline_access",
  "description" : "${role_offline-access}",
  "composite" : false,
  "clientRole" : false,
  "containerId" : "demorealm"
}, {
  "id" : "75faab8b-6460-43d8-a1f1-b4cb2ea840e8",
  "name" : "uma_authorization",
  "description" : "${role_uma_authorization}",
  "composite" : false,
  "clientRole" : false,
  "containerId" : "demorealm"
} ]
```

### Adding roles to users or groups
The command `add-roles` can be used to add a specific role to an existing user
or group in a realm.

The following example adds the **offline_access** role to the user **demouser**
in the **demorealm** realm:

```
$ kcadm.sh add-roles -r demorealm --uusername testuser --rolename offline_access
```

### Removing roles from users or groups
The command `remove-roles` removes an assigned role from a user or group. The
following example removes the role **offline_access** from the user **demouser**
in the **demorealm** realm:

```
$ kcadm.sh remove-roles -r demorealm --uusername demouser --rolename offline_access
```

### Update a user's password
The Admin CLI can be used to update user's passwords. A temporary password can
be passed in order to force the user to updated it at next login.
This is a very useful feature especially inside a scripted automation.
The following example sets a temporary password **redhat** for the user
**demouser** in the **demorealm** realm:

```
$ set-password -r demorealm --username demouser --new-password redhat -t
```

The `-t` option forces the password to be temporary.

## Additional informations
https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html-single/server_administration_guide/index#the_admin_cli
