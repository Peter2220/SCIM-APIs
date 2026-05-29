# Keycloak SCIM Schema Reference

## Retrieve SCIM Schemas

```bash id="5aqjra"
# Retrieve all SCIM schema definitions exposed by Keycloak
curl -k -X GET \
  "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Schemas" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Accept: application/scim+json" | jq
```

---

# Example SCIM Schema Response

```json id="t0h6y1"
{
  "schemas": [
    "urn:ietf:params:scim:api:messages:2.0:ListResponse"
  ],
  "totalResults": 3,
  "startIndex": 1,
  "itemsPerPage": 3
}
```

---

# SCIM Core Group Schema

```json id="b8efch"
{
  "id": "urn:ietf:params:scim:schemas:core:2.0:Group",
  "name": "Group",
  "description": "Group",
  "attributes": [
    {
      "name": "displayName",
      "type": "string",
      "description": "Human-readable group name"
    },
    {
      "name": "members",
      "type": "complex",
      "multiValued": true,
      "description": "List of group members"
    },
    {
      "name": "externalId",
      "type": "string",
      "description": "External identifier mapped from external systems"
    }
  ]
}
```

---

# SCIM Core User Schema

```json id="rjlwmx"
{
  "id": "urn:ietf:params:scim:schemas:core:2.0:User",
  "name": "User",
  "description": "User Account"
}
```

---

# Common User Attributes

```json id="2n5jbd"
{
  "userName": {
    "type": "string",
    "required": true,
    "description": "Unique username for the account"
  },
  "displayName": {
    "type": "string",
    "description": "Display name shown in applications"
  },
  "emails": {
    "type": "complex",
    "multiValued": true,
    "description": "Email addresses associated with the user"
  },
  "active": {
    "type": "boolean",
    "description": "Indicates whether the user account is active"
  },
  "groups": {
    "type": "complex",
    "multiValued": true,
    "description": "Groups the user belongs to"
  },
  "externalId": {
    "type": "string",
    "description": "External identifier from another identity source"
  }
}
```

---

# User Name Object Structure

```json id="5vdf54"
{
  "name": {
    "formatted": "Full formatted name",
    "givenName": "First name",
    "middleName": "Middle name",
    "familyName": "Last name",
    "honorificPrefix": "Prefix",
    "honorificSuffix": "Suffix"
  }
}
```

---

# Enterprise User Extension Schema

```json id="bz4kr7"
{
  "id": "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
  "name": "EnterpriseUser",
  "description": "Enterprise User"
}
```

---

# Enterprise User Attributes

```json id="px9zsu"
{
  "division": {
    "type": "string",
    "description": "Business division"
  },
  "department": {
    "type": "string",
    "description": "Department name"
  },
  "organization": {
    "type": "string",
    "description": "Organization name"
  },
  "employeeNumber": {
    "type": "string",
    "description": "Employee identifier"
  },
  "costCenter": {
    "type": "string",
    "description": "Financial cost center"
  },
  "manager": {
    "type": "complex",
    "description": "Manager relationship object",
    "subAttributes": {
      "value": "Manager user ID",
      "displayName": "Manager display name"
    }
  }
}
```

---

# Recommended SCIM Attribute Mapping Examples

## User Mapping Example

```json id="4qvoww"
{
  "user": {
    "principalName": "userName",
    "displayName": "displayName",
    "userName": "userName",
    "email": "emails",
    "id": "id",
    "givenName": "name.givenName",
    "familyName": "name.familyName"
  }
}
```

---

## Group Mapping Example

```json id="f9qujo"
{
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

---

# Important Notes

```text id="o4tpnj"
# Keycloak SCIM API currently exposes:
# - Core User schema
# - Core Group schema
# - Enterprise User extension schema

# The externalId attribute is commonly used as a stable identifier
# when integrating SCIM groups with external platforms.
```
