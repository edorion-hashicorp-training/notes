https://www.vaultproject.io/docs/concepts/policies
https://learn.hashicorp.com/tutorials/vault/policies?in=vault/policies

- facilitate RBAC
- policies define paths and capabilities for whom or what ever is assigned the policy to access
- practices **least privileged** access
- 2 policies created on initialization *root* and *default*
	- root
		- god mode for the namespace and kids
	- default
		- capabilities to self manage token

### Globs
* is a wild card to match everything after so always last char in a a path match
+ is a ild card to match any word in a path i.e. "secret/+/teamb" matches anything under secret that has a teamb sub folder so cant be the last char in a path match and must be between '/'s


policy example
"*" - `vault namespace list` OK
"/*" - `vault namespace list` OK
"+/*" - `vault namespace list` OK
"/+/*" - `vault namespace list` OK
"sys/+/*" - `vault namespace list` OK

### /sys/
- core config folder


### [[HCL]]
[HCL](https://github.com/hashicorp/hcl)
- can be substituted with JSON


#HCL


### Writing a policy
**GATHER REQUIREMENTS**

Capability | Associated HTTP verbs
-----|-----
create | POST/PUT
read | GET
update | POST/PUT
delete | DELETE
list | LIST
sudo | \-
deny | \-

- deny is the strongest capability care of **least privileged** access 
- listing always operates on a prefix, policies must operate on a prefix because Vault will sanitize request paths to be prefixes.

### sudo
- the sudo capability allows access to [Root protected endpoints](https://learn.hashicorp.com/tutorials/vault/policies?in=vault/policies#root-protected-api-endpoints)

### priority rules
If the same pattern appears in multiple policies, we take the union of the capabilities. If different patterns appear in the applicable policies, we take only the highest-priority match from those policies.
P1 = policy1 P2 = polic2
1.  If the first wildcard (`+`) or glob (`*`) occurs earlier in `P1`, `P1` is lower priority
2.  If `P1` ends in `*` and `P2` doesn't, `P1` is lower priority
3.  If `P1` has more `+` (wildcard) segments, `P1` is lower priority
4.  If `P1` is shorter, it is lower priority
5.  If `P1` is smaller lexicographically, it is lower priority


### Policy Templates
https://www.vaultproject.io/docs/concepts/policies#templated-policies
- Currently `identity` information can be injected, and currently the `path` keys in policies allow injection.
- Where possible / always use ID's as names may duplicate for different entities

Name | Description
-----|-----
`identity.entity.id` | The entity's ID
`identity.entity.name` | The entity's name
`identity.entity.metadata.<metadata key>` | Metadata associated with the entity for the given key
`identity.entity.aliases.<mount accessor>.id` | Entity alias ID for the given mount
`identity.entity.aliases.<mount accessor>.name` | Entity alias name for the given mount
`identity.entity.aliases.<mount accessor>.metadata.<metadata key>` | Metadata associated with the alias for the given mount and metadata key
`identity.groups.ids.<group id>.name` | The group name for the given group ID
`identity.groups.names.<group name>.id` | The group ID for the given group name
`identity.groups.ids.<group id>.metadata.<metadata key>` | Metadata associated with the group for the given key
`identity.groups.names.<group name>.metadata.<metadata key>` | Metadata associated with the group for the given key

#templating 

### Fine-Grained Control
- [`required_parameters`](https://www.vaultproject.io/docs/concepts/policies#required_parameters) A list of parameters that must be specified. "Requires a user to create"
-  [`allowed_parameters`](https://www.vaultproject.io/docs/concepts/policies#allowed_parameters) A list of allowed parameters (including / down to values of keys)
- [`denied_parameters`](https://www.vaultproject.io/docs/concepts/policies#denied_parameters) \- Blacklists a list of parameter and values


### Reponse wrapping TTL
https://www.vaultproject.io/docs/concepts/policies#required-response-wrapping-ttls
To be set when [[Response-Wrapping Tokens]]
