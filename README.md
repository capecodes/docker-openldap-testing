# OpenLDAP integration testing server

### Run a simple server
```bash

# create a network for the ldap testing
docker network create ldap-testing

# run an openldap server
docker run -dt -v openldap-testing-data:/var/lib/ldap --name=openldap-testing --hostname=ldap.example.org --net=ldap-testing -p :389:389 -p :636:636 capecodes/openldap-testing
```

### Test binding various users
```bash

# extract the Docker network IP for the OpenLDAP testing server, this is necessary as we'll need to use it
# in an --add-host for DNS 'ldap.example.org', which is the CN on the testing server certificate. This is
# because the ldap tools (ldapwhoami, ldapsearch etc) do TLS certificate host name verification

ldap_server_ip=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' openldap-testing)

# bind the 'cn=admin,dc=example,dc=org' user using StartTLS
docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  capecodes/openldap-testing \
  ldapwhoami -D 'cn=admin,dc=example,dc=org' -w password -H 'ldap://ldap.example.org' -v -Z

# bind the 'cn=readonly,dc=example,dc=org' user using StartTLS
docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  capecodes/openldap-testing \
  ldapwhoami -D 'cn=readonly,dc=example,dc=org' -w password -H 'ldap://ldap.example.org' -v -Z

# bind the 'cn=someguy,dc=example,dc=org' user using StartTLS
docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  capecodes/openldap-testing \
  ldapwhoami -D 'cn=someguy,dc=example,dc=org' -w password -H 'ldap://ldap.example.org' -v -Z

# bind the 'cn=admin,dc=example,dc=org' user using TLS
docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  capecodes/openldap-testing \
  ldapwhoami -D 'cn=admin,dc=example,dc=org' -w password -H 'ldaps://ldap.example.org' -v

# bind the 'cn=readonly,dc=example,dc=org' user using TLS
docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  capecodes/openldap-testing \
  ldapwhoami -D 'cn=readonly,dc=example,dc=org' -w password -H 'ldaps://ldap.example.org' -v

# bind the 'cn=someguy,dc=example,dc=org' user using TLS
docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  capecodes/openldap-testing \
  ldapwhoami -D 'cn=someguy,dc=example,dc=org' -w password -H 'ldaps://ldap.example.org' -v
```

### Load LDIFs

```bash

# extract the Docker network IP for the OpenLDAP testing server, this is necessary as we'll need to use it
# in an --add-host for DNS 'ldap.example.org', which is the CN on the testing server certificate. This is
# because the ldap tools (ldapwhoami, ldapsearch etc) do TLS certificate host name verification

ldap_server_ip=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' openldap-testing)

docker run -it --rm \
  --net=ldap-testing \
  --entrypoint='' \
  --add-host=ldap.example.org:${ldap_server_ip} \
  -v $(pwd)/data.ldif:/data.ldif \
  capecodes/openldap-testing \
  ldapadd -D 'cn=admin,dc=example,dc=org' -w password -H 'ldap://ldap.example.org' -v -Z -c -f /data.ldif
```
