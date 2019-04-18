# The Admin REST API
Red Hat Single Sign-On offers an administration REST API to implement actions
from script or custom code.

### Obtaining the Access Token
To obtain an OIDC access token for the admin user:
```
curl \
  -d "client_id=admin-cli" \
  -d "username=admin" \
  -d "password=redhat" \
  -d "grant_type=password" \
  "http://localhost:8080/auth/realms/master/protocol/openid-connect/token"
```

The above command produces a json payload like this:

```
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJEM3Bzc0MtRnp1VzNsSndXTFViaDI3aDByUGxBVWtZMFRSS0VMZjZ1VVJFIn0.eyJqdGkiOiJiYjAyZmI4MS0zY2NlLTRiYjctOTVmNC04YzI0NGUyZTMzMDYiLCJleHAiOjE1NTU0NDExMjEsIm5iZiI6MCwiaWF0IjoxNTU1NDQxMDYxLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvbWFzdGVyIiwic3ViIjoiN2Q2NDg4YzAtMWUwNi00NTczLThiZGEtYzcyNjdiODhjMmI4IiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiYWRtaW4tY2xpIiwiYXV0aF90aW1lIjowLCJzZXNzaW9uX3N0YXRlIjoiZjE3OGM4ZDAtY2Q2ZC00MzE2LWIwZWEtNTNlZWU1YjBhMmY1IiwiYWNyIjoiMSIsInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwicHJlZmVycmVkX3VzZXJuYW1lIjoiYWRtaW4ifQ.RCf9RVg7cddZf3yU8VoiVGglNasVl7JtWrkt_o8swLcmJxuVQQ7Iga_38bqZcoHYEiMIdwGmoqCfZ5iftHvUX9L4EwECe2_Eaj6HXxMoc89cRdjJOXsCy7FpzdaB1b_k2LN5Wt6Gs6PeVRVy72dFAGNt5c77Nf8OzrbEEXP-XRh5esZpTI9z68QfO_Be73ynjUDckfiUgI_eqe4ym1kjCmeMFiTAp3S-p9Cs5bDtEf-vib7Hs1giXUQH8nXtK--f5VAV2zyFOZRR612WmJAHuJdsajfCWhA2WZFvwf5YwKkYakdX5o6NwnUfAtZ6OPYmu_wBquncYMssL57pCAAW5g","expires_in":60,"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIyODgxMmVmZi0zNDc4LTRiOGQtYWNiMS04NGM5MTFkMGI1MDAifQ.eyJqdGkiOiIwMzc4MmI5My1hM2VjLTQxMWQtYTM1OC1mYzgyZWJlMWE5ZmYiLCJleHAiOjE1NTU0NDI4NjEsIm5iZiI6MCwiaWF0IjoxNTU1NDQxMDYxLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvbWFzdGVyIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL21hc3RlciIsInN1YiI6IjdkNjQ4OGMwLTFlMDYtNDU3My04YmRhLWM3MjY3Yjg4YzJiOCIsInR5cCI6IlJlZnJlc2giLCJhenAiOiJhZG1pbi1jbGkiLCJhdXRoX3RpbWUiOjAsInNlc3Npb25fc3RhdGUiOiJmMTc4YzhkMC1jZDZkLTQzMTYtYjBlYS01M2VlZTViMGEyZjUiLCJzY29wZSI6ImVtYWlsIHByb2ZpbGUifQ.Q7H4kO20H21yyy63Hduz1-2A0hkkl_ghTSleR9FcmGA","token_type":"bearer","not-before-policy":0,"session_state":"f178c8d0-cd6d-4316-b0ea-53eee5b0a2f5","scope":"email profile"}
```

The access token in the field `access_token` lasts for 60 seconds. It can be
extracted and used to for API calls as a bearer token.

### Executing API calls
The following example is a GET method that shows the configuration of the
**training** realm.

```
$ curl -X GET -H "Authorization: Bearer ${TOKEN}" \
  http://localhost:8080/auth/admin/realms/training
```

Where `${TOKEN}` is a variable containing the temporary token.

### Embedding Admin API in code
The following example is a minimal Python2 script that uses simple `httplib`
module (deprecated in Python 3.x) to get the token and use it in simple GET methods.

```
#!/usr/bin/env python

import sys
import json
import httplib,urllib

def get_token(conn, username, password):
    api_path = '/auth/realms/master/protocol/openid-connect/token'
    params = urllib.urlencode({'client_id': 'admin-cli', 'username': username, 'password': password, 'grant_type': 'password'})
    headers = {"Content-type": "application/x-www-form-urlencoded", "Accept": "text/plain"}

    conn.request("POST", api_path, params, headers)
    response = conn.getresponse()
    if response.status != 200:
        sys.exit(1)

    dict_resp = json.loads(response.read())

    return dict_resp['access_token']

def admin_api(conn, method, api_path, token):
    headers = {"Authorization": "bearer " + token}
    params = ""
    conn.request(method, api_path, params, headers)
    response = conn.getresponse()
    return response.read()


def main():
    admin_user = 'admin'
    admin_password = 'redhat'
    connection = httplib.HTTPConnection('localhost:8080')
    api_basepath = '/auth/admin/realms'

    if len(sys.argv) < 3:
        print "Not enough arguments"
        sys.exit(1)

    # Define the method using the first command line argument
    method = sys.argv[1]
    if method != 'GET':
        print "Only GET method is implemented"
        sys.exit(1)

    # Pass the api endpoint as second command line argument
    api_endpoint = api_basepath + sys.argv[2]

    token = get_token(connection, admin_user, admin_password)
    json_resp = admin_api(connection, method, api_endpoint, token)
    print json_resp

    connection.close()

if __name__ == '__main__':
  main()
```

The above code snippet can be pasted in a python script and executed. The
following example shows a simple GET method to view the users in the **training**
realm:

```
$ python admin_api_demo.py GET /training/users
```

This example produces an output similar to the following:

```
[
    {
        "access": {
            "impersonate": true,
            "manage": true,
            "manageGroupMembership": true,
            "mapRoles": true,
            "view": true
        },
        "createdTimestamp": 1555595472824,
        "disableableCredentialTypes": [],
        "email": "demouser@redhat.com",
        "emailVerified": true,
        "enabled": true,
        "firstName": "John",
        "id": "92e37f6d-1b20-4f34-8c95-f60f1d7a0d6b",
        "lastName": "Doe",
        "notBefore": 0,
        "requiredActions": [
            "UPDATE_PASSWORD",
            "UPDATE_PROFILE"
        ],
        "totp": false,
        "username": "demouser"
    }
]
```

### REST API Documentation
All REST APIs for Red Hat Single Sign-On are documented at this [link](https://access.redhat.com/webassets/avalon/d/red-hat-single-sign-on/version-7.3/restapi/).
