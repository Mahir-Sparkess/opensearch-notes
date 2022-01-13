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
- No xpack permissions.

LDAP configurations already integrated in the YAML files with default settings. No obvious dashboard access; will have to use an IDE to configure the YAML files.

  ## Conclusion

   Better documentation for configuration:
  - - See expandsion to the YAML files section about ROLES, USERS, MAPPINGs with examples and work flow on how to set up.
  - - Look to see updates where configuring OpenSearch is not difficult to figure out and is smooth and intuitive via guides in the documentation.

ROLE/INDEX permissions mapped to connected host/ip:

- - See if there is permissions to configure what ip will be given what role. i.e. IP filtering connected users of a given role.
- - There is a general http ip filtering in the `opensearch_config.yaml` that effects entirety access. Not granular to a specific index or role.

ROLE/INDEX authentication via API Key or Token:

- - See if OpenSearch will add API Key or Token based authentication like ElasticSearch has. Currently searching the docs keywords: "API Key", "Key" or "Token" results back with nothing.
- - This isn't necessary, it can be replaced by USERs mapped to applications rather than API Keys for applications. Rather than a key, you use username and password to authenticate. 

XPACk

- I do not know what this is but there are no xpack related actions. It might be a plugin? Or how relevant this is. 
- Currently there not permissions/actions related to `cluster:monitor/xpack`, `cluster:admin/xpack`, `indices:data/read/xpack`.