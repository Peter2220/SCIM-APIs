# SCIM API Users and Groups Bulk Creation

## Create SCIM Groups

```bash
# List of football clubs to provision as SCIM groups
TEAMS=(
  "Real Madrid"
  "Barcelona"
  "Manchester City"
  "Liverpool"
  "Bayern Munich"
  "PSG"
  "Juventus"
  "Arsenal"
  "AC Milan"
  "Inter Milan"
  "Chelsea"
  "Manchester United"
  "Tottenham"
  "Atletico Madrid"
  "Borussia Dortmund"
  "Napoli"
  "Roma"
  "Ajax"
  "Benfica"
  "Porto"
)

# Create each SCIM group dynamically
for team in "${TEAMS[@]}"; do

  # Print current group being provisioned
  echo ">>> Provisioning club: $team"

  # Create the SCIM group
  curl -k -X POST \
    "${KEYCLOAK_URL}/realms/${REALM_NAME}/scim/v2/Groups" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H "Content-Type: application/scim+json" \
    -H "Accept: application/scim+json" \
    -d "{
      \"schemas\": [
        \"urn:ietf:params:scim:schemas:core:2.0:Group\"
      ],
      \"displayName\": \"$team\"
    }"

  # Print separator line
  echo -e "\n--------------------------------------------------\n"

done
```

---

# Create Users via Keycloak Admin API and Add Them to Groups

```bash
# Player roster format:
# username:givenName:familyName:email:groupName

PLAYERS=(
  "vini_jr:Vinicius:Junior:vinicius@realmadrid.com:Real Madrid"
  "k_mbappe:Kylian:Mbappe:k.mbappe@realmadrid.com:Real Madrid"
  "j_bellingham:Jude:Bellingham:jude@realmadrid.com:Real Madrid"
  "rodrygo:Rodrygo:Goes:rodrygo@realmadrid.com:Real Madrid"
  "courtois:Thibaut:Courtois:courtois@realmadrid.com:Real Madrid"

  "r_lewandowski:Robert:Lewandowski:robert@barcelona.com:Barcelona"
  "l_yamal:Lamine:Yamal:lamine@barcelona.com:Barcelona"
  "p_pedri:Pedro:Gonzalez:pedri@barcelona.com:Barcelona"
  "gavi:Pablo:Gavi:gavi@barcelona.com:Barcelona"
  "r_araujo:Ronald:Araujo:araujo@barcelona.com:Barcelona"

  "e_haaland:Erling:Haaland:erling@mancity.com:Manchester City"
  "k_debruyne:Kevin:DeBruyne:kevin@mancity.com:Manchester City"
  "p_foden:Phil:Foden:phil@mancity.com:Manchester City"
  "j_grealish:Jack:Grealish:grealish@mancity.com:Manchester City"
  "r_dias:Ruben:Dias:dias@mancity.com:Manchester City"

  "m_salah:Mohamed:Salah:msalah@liverpool.com:Liverpool"
  "v_vandijk:Virgil:VanDijk:virgil@liverpool.com:Liverpool"
  "d_nunez:Darwin:Nunez:darwin@liverpool.com:Liverpool"
  "a_macallister:Alexis:MacAllister:alexis@liverpool.com:Liverpool"
  "t_arnold:Trent:AlexanderArnold:trent@liverpool.com:Liverpool"

  "h_kane:Harry:Kane:hkane@bayern.com:Bayern Munich"
  "j_musiala:Jamal:Musiala:jamal@bayern.com:Bayern Munich"
  "l_sane:Leroy:Sane:sane@bayern.com:Bayern Munich"
  "m_de_ligt:Matthijs:DeLigt:deligt@bayern.com:Bayern Munich"
  "k_kimmich:Joshua:Kimmich:kimmich@bayern.com:Bayern Munich"

  "o_dembele:Ousmane:Dembele:ousmane@psg.com:PSG"
  "b_barcola:Bradley:Barcola:bradley@psg.com:PSG"
  "m_skriniar:Milan:Skriniar:skriniar@psg.com:PSG"
  "g_donnarumma:Gianluigi:Donnarumma:donna@psg.com:PSG"
  "m_ugarte:Manuel:Ugarte:ugarte@psg.com:PSG"
)

# Pull all existing groups once and cache locally
echo ">>> Fetching group catalog from Keycloak..."

ALL_GROUPS_JSON=$(curl -s -k -X GET \
  "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/groups" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

echo ">>> Starting user provisioning..."

for player in "${PLAYERS[@]}"; do

  # Extract username
  USERNAME=$(echo "$player" | cut -d: -f1)

  # Extract given name
  GIVEN=$(echo "$player" | cut -d: -f2)

  # Extract family name
  FAMILY=$(echo "$player" | cut -d: -f3)

  # Extract email
  EMAIL=$(echo "$player" | cut -d: -f4)

  # Extract target group
  GROUP_NAME=$(echo "$player" | cut -d: -f5)

  echo ">>> Creating user: $USERNAME"

  # Create the user
  curl -s -k -X POST \
    "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/users" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H "Content-Type: application/json" \
    -d "{
      \"username\": \"$USERNAME\",
      \"firstName\": \"$GIVEN\",
      \"lastName\": \"$FAMILY\",
      \"email\": \"$EMAIL\",
      \"enabled\": true,
      \"emailVerified\": true,
      \"credentials\": [
        {
          \"type\": \"password\",
          \"value\": \"12345\",
          \"temporary\": false
        }
      ]
    }"

  # Retrieve the newly created user ID
  USER_ID=$(curl -s -k -X GET \
    "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/users?username=${USERNAME}&exact=true" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    | jq -r '.[0].id // empty')

  # Retrieve matching group ID
  GROUP_ID=$(echo "$ALL_GROUPS_JSON" \
    | jq -r --arg gname "$GROUP_NAME" \
    '.[] | select(.name == $gname) | .id' \
    | head -n 1)

  # Validate IDs before mapping
  if [ -n "$USER_ID" ] && [ -n "$GROUP_ID" ]; then

    echo ">>> Adding $USERNAME to group: $GROUP_NAME"

    # Add user to group
    curl -s -k -X PUT \
      "${KEYCLOAK_URL}/admin/realms/${REALM_NAME}/users/${USER_ID}/groups/${GROUP_ID}" \
      -H "Authorization: Bearer ${ACCESS_TOKEN}" \
      -H "Content-Length: 0"

    echo ">>> Successfully mapped $USERNAME -> $GROUP_NAME"

  else

    echo ">>> Failed to map user/group"
    echo "USER_ID=${USER_ID:-NOT_FOUND}"
    echo "GROUP_ID=${GROUP_ID:-NOT_FOUND}"

  fi

  echo "--------------------------------------------------"

done
```
