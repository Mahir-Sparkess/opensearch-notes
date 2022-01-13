# January 2022 notes

## Background
Should we be paying for ReadOnlyRest (ROR) elasticsearch security filter?

Other options:
- Vanilla Elasticsearch
- Opensearch (Opensource fork of Elasticsearch)

Desired features:
- IP Filtering
  - Allows us to create internal indices
  - Restricts the scope of potential logins
  - Can restrict certain priviledged access from a single host e.g. Kibana
- LDAP
  - Easy user management using existing mechanisms
- API Keys
  - Less management for application access than username:password
- Restrict admin access to particular users using a specified network
- Restrict write access from applications within a specified network and to specified indices
- Read access for specific indices internally
- Read access for specific indices globally

Reasons for changing:
- Save the annual fee
- Simplify deployment
- Reduced lag from new elastic release to upgrade as don't need to wait for ROR to be updated.

## Vanilla Elasticsearch

### Pros:
- Free
- Well documented
- Decreased deployment and maintenance complexity

### Cons:
- Missing IP filtering and LDAP connection in free tier

We make use of IP filtering to reduce the scope of allowed incoming connections for particular actions and indices.
We make use of LDAP to reduce the user management burden.
If we can be happy with separate user management and no IP filtering this is a good choice. It would mean we just drop the 
security filter and apply Elastic natives security instead. Might require a short period of downtime while the new security rules 
are applied.

## Amazon's Opensearch

### Set up with docker

- Easy
- Run docker and compose up the OpenSearch' `docker-compose.yml`

### Configuring OpenSearch

- Difficult
- Documentation is unhelpful, required a lot of experimenting of things manually to figure things out.
- Vanilla elasticsearch docs are helpful, for now. It will depend on how far they diverge over time whether that remains useful.
- Security examples in documentation are for whitelisting but this is not the recommended approach. 
- Documentation does not mention being able to configure via Dashboard therefore no information on how to do so either.

#### Configure via YAML

- Use VScode or other VM accessing tool to edit YAML files (VScode is nice to work with containers using an IDE):
  - Security (Authentication and Authorization): `usr/share/opensearch/plugins/opensearch-security/securityconfig`
  - [YAML config information](https://opensearch.org/docs/latest/security-plugin/configuration/yaml/): Not a lot of useful information, whitelisting is easy but not recommended. Maybe requires maturing for examples. 
  - Recommends creating Roles, Users and mapping them to configure authc and authz.

### Configure via Dashboard

- Use Dashboard at `localhost:5601` and access configuration settings under: `Home > OpenSearch Plugins > Security` 
- Dashboard is the Kibana equivalent.
  - This is equivalent to above but with an interface, skipping the IDE.
  - First Create a USER (type, name, passwords) then Create a ROLE. Manage mapping to add USERs to that ROLE. ROLES can be mapped to indices.
  - USER-INDEX permissions are dictated via the ROLE.
  - Intuitively it should be the otherway round but no obvious path that way as far as I could see.

OpenSearch Equivalents to ElasticSearch Settings:

- `data_access` = `indices:data/*`, `indices:admin/mappings/fields/get*`, `indices:admin/resolve/index`, `indices:admin/mapping/put`
- `manage_aliases` = `indices:admin/aliases*`

LDAP configurations already integrated in the YAML files with default settings. No obvious dashboard access; will have to use an IDE to configure the YAML files.

### Pros:
- Free
- Opensource

### Cons:
- Documentation is lacking high-quality examples
- May diverge from core elastic stack
- Would require a total migration
- Missing some features we make use of

When Richard reviewed this 2 years ago, it was clunky and difficult to set up the security filters. The interface has been tidied up and improved but the documentation is still far behind the fantastic documentation provided by ReadOnlyRest and Elastic.

There are some missing features, API key authentication and source IP filtering. We make use of API keys for managing application iteractions. Having separate API keys for each app makes it easy to separate out activity in the logs and allows you to debug issues with authentication easily using:
`tail -f /var/log/elasticsearch/<logname> | grep <API KEY>`.

### What is needed?
Better documentation for configuration:
  - See expandsion to the YAML files section about ROLES, USERS, MAPPINGs with examples and work flow on how to set up.
  - Look to see updates where configuring OpenSearch is not difficult to figure out and is smooth and intuitive via guides in the documentation.

ROLE/INDEX permissions restricted by host/ip:
- See if there is permissions to configure what ip will be given what role. i.e. IP filtering connected users of a given role.
- There is a general http ip filtering in the `opensearch_config.yaml` that effects entirety access. Not granular to a specific index or role.

ROLE/INDEX authentication via API Key or Token:
- See if OpenSearch will add API Key or Token based authentication like ElasticSearch has. Currently searching the docs keywords: "API Key", "Key" or "Token" results back with nothing.
- This isn't necessary, it can be replaced by USERs mapped to applications rather than API Keys for applications. Rather than a key, you use username and password to authenticate. 

## Conclusion (Richard)

I would be hesitant to move to using Amazon's Opensearch, unless there was a policy need for true opensource. The RBAC (Role Based Access Control) is fiddly and complex and (at present) poorly documented. Given the ease of configuration we currently have, it would be a step back. I would also be concered about the ability of the opensource community to keep pace with the innovation of Elastic. Divergence would mean that the fantastic documentation from Elastic, which largely is applicable to Opensearch, may no longer be relevant and we would have to rely on more patchy docs.

The vanllia Elastic offering might be worth a look because we should be able to turn off ROR and implement some pre-tested policies with a small downtime. With this approach, we lose the LDAP user management. Just becomes one more username and password to remember (keep in your password manager). We would also lose the IP filtering. It would have to be decided if we could create the necessary protection for our data without this added restriction.