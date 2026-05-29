# Keycloak SCIM API Configuration and Usage

## Define Environment Variables

```bash
# Base Keycloak URL used throughout the script and examples
KEYCLOAK_URL="https://keycloak.example.com"

# Target realm where SCIM is enabled
REALM_NAME="test"

# SCIM client identifier
CLIENT_ID="scim-client"

# SCIM client secret
CLIENT_SECRET="secret"
```

---

# Authenticate to the Keycloak Admin CLI

```bash
# Authenticate to the Keycloak Admin CLI using the master realm administrator account
/opt/keycloak/bin/kcadm.sh config credentials \
  --server http://localhost:8080 \
  --realm master \
  --user admin
```

---

# Enable the SCIM API

```bash
# Enable the experimental SCIM API for the target realm
/opt/keycloak/bin/kcadm.sh update realms/${REALM_NAME} \
  -s scimApiEnabled=true
```

```bash
# Verify that the SCIM API is enabled for the realm
/opt/keycloak/bin/kcadm.sh get realms/${REALM_NAME} | grep scim
```

---

# Create a SCIM Service Client

```bash
# Create a confidential client with service account access enabled
/opt/keycloak/bin/kcadm.sh create clients -r ${REALM_NAME} \
  -s clientId=${CLIENT_ID} \
  -s serviceAccountsEnabled=true \
  -s publicClient=false \
  -s secret=${CLIENT_SECRET}
```

---

# Grant Required Roles to the Service Account

```bash
# Grant the "manage-users" role to the SCIM client service account
/opt/keycloak/bin/kcadm.sh add-roles -r ${REALM_NAME} \
  --uusername service-account-${CLIENT_ID} \
  --cclientid realm-management \
  --rolename manage-users
```

```bash
# Grant the "view-realm" role to the SCIM client service account
/opt/keycloak/bin/kcadm.sh add-roles -r ${REALM_NAME} \
  --uusername service-account-${CLIENT_ID} \
  --cclientid realm-management \
  --rolename view-realm
```

---

# Retrieve an Access Token

```bash
# Request an OAuth access token using the client credentials grant type
ACCESS_TOKEN=$(curl -ks -X POST "${KEYCLOAK_URL}/realms/${REALM_NAME}/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}" \
  | jq -r .access_token)
```
---

# Create a SCIM User

```bash
# Create a SCIM user named "jdoe"
curl -vk -X POST "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Users" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/scim+json" \
  -H "Accept: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "userName": "jdoe",
    "name": {
      "givenName": "John",
      "familyName": "Doe"
    },
    "displayName": "John Doe",
    "emails": [
      {
        "value": "jdoe@example.com"
      }
    ]
  }'
```

---

# Create a SCIM Group

```bash
# Create a SCIM group named "mygroup"
curl -vk -X POST "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Groups" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/scim+json" \
  -H "Accept: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "displayName": "mygroup"
  }'
```

---

# Add a User to a Group

```bash
# Add the user "jdoe" to the specified SCIM group
curl -vk -X PATCH "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Groups/mygroup" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/scim+json" \
  -H "Accept: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
      {
        "op": "add",
        "path": "members",
        "value": [
          {
            "value": "jdoe"
          }
        ]
      }
    ]
  }'
```

---

# Retrieve SCIM Metadata

```bash
# Retrieve the SCIM Service Provider configuration
curl -vk -X GET "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/ServiceProviderConfig" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

```bash
# Retrieve all SCIM schema definitions
curl -vk -X GET "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Schemas" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

```bash
# Retrieve all SCIM resource types
curl -vk -X GET "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/ResourceTypes" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

---

# Retrieve SCIM Groups

```bash
# Query all SCIM groups from the realm
curl -k -X GET "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Groups" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Accept: application/scim+json"
```
# SCIM API Validation and Troubleshooting

## Validate SCIM Service Provider Configuration

```bash
curl -k -X GET \
  "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/ServiceProviderConfig" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq
```

Expected result:

* `"filter":{"supported":true}`
* SCIM endpoints available
* OAuth bearer authentication enabled
---

# Retrieve Single SCIM Group by ID

Example:

```bash
curl -k -X GET \
  "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Groups/a6d0ff9c-e747-400c-b3ee-c5469498205b" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq
```

---

# Retrieve Group Members via Admin API

Example:

```bash
curl -k -X GET \
  "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/groups/a6d0ff9c-e747-400c-b3ee-c5469498205b/members" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq
```

```bash
FILTER='displayName eq "Arsenal"'

curl -k -G \
  "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Groups" \
  --data-urlencode "filter=${FILTER}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq
```
---
# Bulk User-to-Group Assignment Script

```bash
# Define mappings between usernames and groups
ROSTER_MAPPINGS=(
  "vini_jr:Real Madrid"
  "k_mbappe:Real Madrid"
  "j_bellingham:Real Madrid"
  "r_lewandowski:Barcelona"
  "l_yamal:Barcelona"
  "p_pedri:Barcelona"
  "e_haaland:Manchester City"
  "k_debruyne:Manchester City"
  "p_foden:Manchester City"
  "m_salah:Liverpool"
  "v_vandijk:Liverpool"
  "h_kane:Bayern Munich"
  "j_musiala:Bayern Munich"
  "o_dembele:PSG"
  "b_barcola:PSG"
  "d_vlahovic:Juventus"
  "b_saka:Arsenal"
  "m_odegaard:Arsenal"
  "r_leao:AC Milan"
  "l_martinez:Inter Milan"
)

# Retrieve all groups from Keycloak and cache them locally
echo "Fetching master group list from Keycloak..."

ALL_GROUPS_JSON=$(curl -s -k -X GET \
  "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/groups" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

# Iterate through each mapping
for mapping in "${ROSTER_MAPPINGS[@]}"; do

  # Extract the username
  USERNAME=$(echo "$mapping" | cut -d: -f1)

  # Extract the group name
  GROUP_NAME=$(echo "$mapping" | cut -d: -f2)

  # Print the current mapping
  echo ">>> Mapping: ${USERNAME} -> ${GROUP_NAME}"

  # Retrieve the user ID dynamically
  USER_ID=$(curl -s -k -X GET \
    "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/users?username=${USERNAME}&exact=true" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    | jq -r '.[0].id // empty')

  # Retrieve the matching group ID from the cached group list
  GROUP_ID=$(echo "${ALL_GROUPS_JSON}" | jq -r \
    --arg gname "${GROUP_NAME}" \
    '.[] | select(.name | contains($gname)) | .id' \
    | head -n 1)

  # Validate that both IDs were found
  if [ -n "${USER_ID}" ] && [ -n "${GROUP_ID}" ]; then

    # Add the user to the group
    curl -k -s -o /dev/null -w "%{http_code}" -X PUT \
      "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/users/${USER_ID}/groups/${GROUP_ID}" \
      -H "Authorization: Bearer ${ACCESS_TOKEN}" \
      -H "Content-Length: 0"

    # Print a success message
    echo "Successfully added ${USERNAME} to ${GROUP_NAME}."

  else

    # Print an error if lookup failed
    echo "Error: ID missing. User (${USERNAME}): ${USER_ID:-NOT FOUND} | Group (${GROUP_NAME}): ${GROUP_ID:-NOT FOUND}"

  fi

  # Print separator for readability
  echo "--------------------------------------------------"

done
```
