# SCIM API Users and Groups Bulk Creation

## Create SCIM Groups

```bash id="z4r2kp"
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

# Create Users via Keycloak Admin API

```bash id="c8pk3a"
# Player roster format:
# username:givenName:familyName:email

PLAYERS=(
  "vini_jr:Vinicius:Junior:vinicius@realmadrid.com"
  "k_mbappe:Kylian:Mbappe:k.mbappe@realmadrid.com"
  "j_bellingham:Jude:Bellingham:jude@realmadrid.com"
  "rodrygo:Rodrygo:Goes:rodrygo@realmadrid.com"
  "courtois:Thibaut:Courtois:courtois@realmadrid.com"

  "r_lewandowski:Robert:Lewandowski:robert@barcelona.com"
  "l_yamal:Lamine:Yamal:lamine@barcelona.com"
  "p_pedri:Pedro:Gonzalez:pedri@barcelona.com"
  "gavi:Pablo:Gavi:gavi@barcelona.com"
  "r_araujo:Ronald:Araujo:araujo@barcelona.com"

  "e_haaland:Erling:Haaland:erling@mancity.com"
  "k_debruyne:Kevin:DeBruyne:kevin@mancity.com"
  "p_foden:Phil:Foden:phil@mancity.com"
  "j_grealish:Jack:Grealish:grealish@mancity.com"
  "r_dias:Ruben:Dias:dias@mancity.com"

  "m_salah:Mohamed:Salah:msalah@liverpool.com"
  "v_vandijk:Virgil:VanDijk:virgil@liverpool.com"
  "d_nunez:Darwin:Nunez:darwin@liverpool.com"
  "a_macallister:Alexis:MacAllister:alexis@liverpool.com"
  "t_arnold:Trent:AlexanderArnold:trent@liverpool.com"

  "h_kane:Harry:Kane:hkane@bayern.com"
  "j_musiala:Jamal:Musiala:jamal@bayern.com"
  "l_sane:Leroy:Sane:sane@bayern.com"
  "m_de_ligt:Matthijs:DeLigt:deligt@bayern.com"
  "k_kimmich:Joshua:Kimmich:kimmich@bayern.com"

  "o_dembele:Ousmane:Dembele:ousmane@psg.com"
  "b_barcola:Bradley:Barcola:bradley@psg.com"
  "m_skriniar:Milan:Skriniar:skriniar@psg.com"
  "g_donnarumma:Gianluigi:Donnarumma:donna@psg.com"
  "m_ugarte:Manuel:Ugarte:ugarte@psg.com"

  "d_vlahovic:Dusan:Vlahovic:dusan@juventus.com"
  "f_chiesa:Federico:Chiesa:chiesa@juventus.com"
  "a_rabiot:Adrien:Rabiot:rabiot@juventus.com"
  "b_szczesny:Wojciech:Szczesny:szczesny@juventus.com"
  "d_bremer:Gleison:Bremer:bremer@juventus.com"

  "b_saka:Bukayo:Saka:saka@arsenal.com"
  "m_odegaard:Martin:Odegaard:martin@arsenal.com"
  "d_rice:Declan:Rice:rice@arsenal.com"
  "g_jesus:Gabriel:Jesus:jesus@arsenal.com"
  "w_saliba:William:Saliba:saliba@arsenal.com"

  "r_leao:Rafael:Leao:rafael@acmilan.com"
  "t_hernandez:Theo:Hernandez:theo@acmilan.com"
  "f_tomori:Fikayo:Tomori:tomori@acmilan.com"
  "o_giroud:Olivier:Giroud:giroud@acmilan.com"
  "m_maignan:Mike:Maignan:maignan@acmilan.com"

  "l_martinez:Lautaro:Martinez:lautaro@inter.com"
  "n_barella:Nicolo:Barella:barella@inter.com"
  "a_bastoni:Alessandro:Bastoni:bastoni@inter.com"
  "h_mkhitaryan:Henrikh:Mkhitaryan:miki@inter.com"
  "m_thuram:Marcus:Thuram:thuram@inter.com"

  "c_palmer:Cole:Palmer:palmer@chelsea.com"
  "r_james:Reece:James:reece@chelsea.com"
  "e_fernandez:Enzo:Fernandez:enzo@chelsea.com"

  "b_fernandes:Bruno:Fernandes:bruno@manutd.com"
  "m_mount:Mason:Mount:mount@manutd.com"
  "r_hojlund:Rasmus:Hojlund:hojlund@manutd.com"

  "h_kane2:Harry:Kane:harry@tottenham.com"
  "s_son:HeungMin:Son:son@tottenham.com"
  "j_maddison:James:Maddison:maddison@tottenham.com"

  "a_griezmann:Antoine:Griezmann:griezmann@atletico.com"
  "a_morata:Alvaro:Morata:morata@atletico.com"
  "koke:Jorge:Resurreccion:koke@atletico.com"

  "j_brandt:Julian:Brandt:brandt@dortmund.com"
  "n_fullkrug:Niclas:Fullkrug:fullkrug@dortmund.com"
  "e_can:Emre:Can:ecan@dortmund.com"

  "v_osimhen:Victor:Osimhen:osimhen@napoli.com"
  "k_kvaratskhelia:Khvicha:Kvaratskhelia:kvara@napoli.com"
  "s_lobotka:Stanislav:Lobotka:lobotka@napoli.com"

  "p_dybala:Paulo:Dybala:dybala@roma.com"
  "l_pellegrini:Lorenzo:Pellegrini:pellegrini@roma.com"
  "r_lukaku:Romelu:Lukaku:lukaku@roma.com"

  "s_bergwijn:Steven:Bergwijn:bergwijn@ajax.com"
  "b_brobbey:Brian:Brobbey:brobbey@ajax.com"
  "j_hato:Jorrel:Hato:hato@ajax.com"

  "a_silva:Antonio:Silva:silva@benfica.com"
  "j_neves:Joao:Neves:neves@benfica.com"
  "r_dias2:Ruben:Dias:dias@benfica.com"

  "p_costa:Diogo:Costa:costa@porto.com"
  "pepe:Kleper:Pepe:pepe@porto.com"
  "e_gonzalez:Evanilson:Gonzalez:evan@porto.com"
)

# Create all users dynamically
for player in "${PLAYERS[@]}"; do

  # Extract username
  USERNAME=$(echo "$player" | cut -d: -f1)

  # Extract given name
  GIVEN=$(echo "$player" | cut -d: -f2)

  # Extract family name
  FAMILY=$(echo "$player" | cut -d: -f3)

  # Extract email address
  EMAIL=$(echo "$player" | cut -d: -f4)

  # Print current user being provisioned
  echo ">>> Registering verified player: $GIVEN $FAMILY ($USERNAME)"

  # Create the user using the Keycloak Admin API
  curl -k -X POST \
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

  # Print separator line
  echo -e "\n--------------------------------------------------\n"

done
```
