https://learn.hashicorp.com/tutorials/vault/policies?in=vault/policies

- facilitate RBAC
- policies define paths and capabilities for whom or what ever is assigned the policy to access
- practices **least privileged** access
- 2 policies created on initialization *root* and *default*
	- root
		- god mode for the namespace and kids
	- default
		- capabilities to self manage token

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

- the sudo capability allows access to [Root protected endpoints](https://learn.hashicorp.com/tutorials/vault/policies?in=vault/policies#root-protected-api-endpoints)

- deny is the strongest capability care of **least privileged** access 
- 