# CP4BA OIDC Client Registration

## Define Environment Variables

```bash id="7j3a0u"
# Base Keycloak URL
KEYCLOAK_URL="https://keycloak.example.com"

# Keycloak realm name
REALM_NAME="test"

# CP4BA OIDC client ID
OIDC_CLIENT_ID="cp4ba"

# CP4BA OIDC client secret
OIDC_CLIENT_SECRET="replace-me"

# SCIM client ID
SCIM_CLIENT_ID="scim-client"

# SCIM client secret
SCIM_CLIENT_SECRET="replace-me"
```

---

# Retrieve CP4BA IAM Credentials from OpenShift

```bash id="xj9r0n"
# Retrieve the IAM administrator username from the platform-auth-idp-credentials secret
iamadmin=$(oc get secret platform-auth-idp-credentials \
  -o jsonpath='{.data.admin_username}' | base64 -d)

# Retrieve the IAM administrator password from the platform-auth-idp-credentials secret
iampass=$(oc get secret platform-auth-idp-credentials \
  -o jsonpath='{.data.admin_password}' | base64 -d)

# Retrieve the CP4BA console route hostname dynamically from OpenShift
iamhost=https://$(oc get route cp-console \
  -o jsonpath="{.spec.host}")

# Request an IAM access token using the password grant flow
iamaccesstoken=$(curl -sk -X POST \
  -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" \
  -d "grant_type=password&username=$iamadmin&password=$iampass&scope=openid" \
  $iamhost/idprovider/v1/auth/identitytoken | jq -r .access_token)
```

---

# Create OIDC + SCIM Identity Provider

```bash id="ukl36v"
# Create a Keycloak OIDC identity provider with SCIM integration enabled
curl --insecure -X POST "${iamhost}/idprovider/v3/auth/idsource/" \
  --header "Authorization: Bearer ${iamaccesstoken}" \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data-raw '{
    "name": "cp4ba",
    "protocol": "oidc",
    "type": "keycloak",
    "description": "cp4ba",
    "enabled": true,
    "idp_config": {
      "client_id": "'"${OIDC_CLIENT_ID}"'",
      "client_secret": "'"${OIDC_CLIENT_SECRET}"'",
      "discovery_url": "'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/.well-known/openid-configuration",
      "token_attribute_mappings": {
        "sub": "sub",
        "email": "email",
        "groups": "groups",
        "subject": "sub",
        "given_name": "given_name",
        "displayName": "full name",
        "family_name": "family_name",
        "preferred_username": "preferred_username",
        "uniqueSecurityName": "preferred_username"
      }
    },
    "scim_config": {
      "scim_base_path": "'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/scim/v2/",
      "grant_type": "client_credentials",
      "token_url": "'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/protocol/openid-connect/token",
      "client_id": "'"${SCIM_CLIENT_ID}"'",
      "client_secret": "'"${SCIM_CLIENT_SECRET}"'",
      "scim_attribute_mappings": {
        "user": {
          "principalName": "userName",
          "displayName": "displayName",
          "userName": "userName",
          "email": "emails",
          "id": "id",
          "givenName": "name.givenName",
          "familyName": "name.familyName"
        },
        "group": {
          "principalName": "displayName",
          "id": "externalId",
          "uid": "externalId",
          "displayName": "displayName",
          "name": "displayName",
          "members": "members"
        }
      }
    }
  }'
```

---

# Retrieve All Identity Providers

```bash id="qjogci"
# Retrieve all configured identity providers
curl -k -X GET "${iamhost}/idprovider/v3/auth/idsource" \
  --header "Authorization: Bearer ${iamaccesstoken}"
```

---

# Retrieve OIDC Provider Configuration

```bash id="77xjlwm"
# Identity provider UID
OIDC_UID="replace-with-uid"

# Retrieve the configuration of a specific identity provider
curl --insecure -X GET \
  "${iamhost}/idprovider/v3/auth/idsource/${OIDC_UID}" \
  --header "Authorization: Bearer ${iamaccesstoken}" \
  --header 'Accept: application/json' \
  | jq
```

---

# Disable an OIDC Identity Provider

```bash id="67pn9u"
# Identity provider UID
OIDC_UID="replace-with-uid"

# Disable an existing OIDC identity provider
curl -k -X PUT \
  "${iamhost}/idprovider/v3/auth/idsource/${OIDC_UID}" \
  --header "Authorization: Bearer ${iamaccesstoken}" \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "name":"cp4ba",
    "protocol":"oidc",
    "type":"keycloak",
    "description":"cp4ba",
    "enabled":false,
    "idp_config":{
      "client_id":"'"${OIDC_CLIENT_ID}"'",
      "discovery_url":"'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/.well-known/openid-configuration",
      "token_attribute_mappings":{
        "sub":"sub",
        "email":"email",
        "groups":"groups",
        "subject":"sub",
        "given_name":"given_name",
        "displayName":"name",
        "family_name":"family_name",
        "preferred_username":"preferred_username",
        "uniqueSecurityName":"preferred_username"
      }
    },
    "scim_config":{
      "client_id":"'"${SCIM_CLIENT_ID}"'",
      "token_url":"'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/protocol/openid-connect/token",
      "grant_type":"client_credentials",
      "scim_base_path":"'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/scim/v2/",
      "scim_attribute_mappings":{
        "user":{
          "email":"emails",
          "givenName":"name.givenName",
          "familyName":"name.familyName",
          "displayName":"userName",
          "principalName":"userName"
        },
        "group":{
          "id":"displayName",
          "principalName":"displayName"
        }
      }
    },
    "jit":true
  }'
```

---

# Create OIDC Identity Provider with JIT Only

```bash id="v5oc6u"
# Create a Keycloak OIDC identity provider using Just-In-Time provisioning only
# No SCIM configuration is used in this example
curl --insecure -X POST "${iamhost}/idprovider/v3/auth/idsource/" \
  --header "Authorization: Bearer ${iamaccesstoken}" \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data-raw '{
    "protocol": "oidc",
    "name": "cp4ba",
    "type": "keycloak",
    "description": "cp4ba-jit-no-scim",
    "enabled": true,
    "jit": true,
    "idp_config": {
      "client_id": "'"${OIDC_CLIENT_ID}"'",
      "client_secret": "'"${OIDC_CLIENT_SECRET}"'",
      "discovery_url": "'"${KEYCLOAK_URL}"'/realms/'"${REALM_NAME}"'/.well-known/openid-configuration",
      "token_attribute_mappings": {
        "sub": "sub",
        "email": "email",
        "groups": "groups",
        "subject": "sub",
        "given_name": "given_name",
        "displayName": "name",
        "family_name": "family_name",
        "preferred_username": "preferred_username",
        "uniqueSecurityName": "preferred_username"
      }
    }
  }'
```

---

# Recommended SCIM Attribute Mappings

```json id="ktgxx2"
{
  "user": {
    "principalName": "userName",
    "displayName": "displayName",
    "userName": "userName",
    "email": "emails",
    "id": "id",
    "givenName": "name.givenName",
    "familyName": "name.familyName"
  },
  "group": {
    "principalName": "displayName",
    "id": "externalId",
    "uid": "externalId",
    "displayName": "displayName",
    "name": "displayName",
    "members": "members"
  }
}
```
