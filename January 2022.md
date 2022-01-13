# January 2022 notes

## Set up with docker

- Easy
- Run docker and compose up the OpenSearch' `docker-compose.yml`

## Configuring OpenSearch

- Difficult
- No APIKey or Token based authorization.
- Documentation is unhelpful, required a lot of experimenting of things manually to figure things out.
- Can not see configuration to add host access permissions; could be in the `opensearch-config.yaml` under `http` or `ldap` but not for individual ROLES or index.
- Documentation does not mention being able to configure via Dashboard therefore no information on how to do so either.

### Configure via YAML

- Use VScode or other VM accessing tool to edit YAML files (VScode is nice to work with containers using an IDE):
- - Security (Authentication and Authorization): `usr/share/opensearch/plugins/opensearch-security/securityconfig`
- - [YAML config information](https://opensearch.org/docs/latest/security-plugin/configuration/yaml/): Not a lot of useful information, whitelisting is easy but not reccomended. Maybe requires maturing for examples. 
- Reccomends creating Roles, Users and mapping them to configure authc and authz.

### Configure via Dashboard

- Use Dashboard at `localhost:5601` and access configuration settings under: `Home > OpenSearch Plugins > Security` 
- - This is equivalent to above but with an interface, skipping the IDE.
- - First Create a USER (type, name, passwords) then Create a ROLE. Manage mapping to add USERs to that ROLE. ROLES can be mapped to indices.
- - USER-INDEX permissions is dictated via the ROLE.
- - Intuitively it should be the otherway round but no obvious path that way as far as I could see.

OpenSearch Equivalents to ElasticSearch Settings:

- `data_access` = `indices:data/*`, `indices:admin/mappings/fields/get*`, `indices:admin/resolve/index`, `indices:admin/mapping/put`
- `manage_aliases` = `indices:admin/aliases*`

LDAP configurations already integrated in the YAML files with default settings. No obvious dashboard access; will have to use an IDE to configure the YAML files.

  ## Continue looking

  - Better configuration docs
  - ROLE/Index host and ip permissions.
  - ROLE/Index mapped to API key/Token (instead of a USER)