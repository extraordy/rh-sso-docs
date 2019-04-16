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

The access token in the first field lasts for 60 seconds and can be used to for
an API call.

### Executing API calls
The following example is a GET method that shows the configuration of the
**training** realm.

```
$ curl -X GET -H "Authorization: Bearer <ACCESS_TOKEN>" \
  http://localhost:8080/auth/admin/realms/training
```

### REST API Documentation
All REST APIs for Red Hat Single Sign-On are documented at this [link](https://access.redhat.com/webassets/avalon/d/red-hat-single-sign-on/version-7.3/restapi/). 
