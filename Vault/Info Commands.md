##### list auth methods
vault auth list --detailed

##### list secrets engines
vault secrets list --detailed


 - 'vault token lookup'
- 'vault token capabilities /sys/namespaces'
- `for v in $(vault policy list); do echo "#### $v"; vault policy read "$v"; done`
- `echo $VAULT_NAMESPACE`
- `vault namespace list -namespace=<insert name of namespace>` 