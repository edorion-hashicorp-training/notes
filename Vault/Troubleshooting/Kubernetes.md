# Kubernetes

#### get vault #pods (nodes)
`kubectl get pods`

#### get single pod status
- using name from 'kubectl get pods'
`kubectl exec vault-0 -- vault status`

#### multi pod status
`for i in {0..2} ; do kubectl exec vault-$i -- vault status ; done`
note: 0-2 =3 iterations

#### #Helm #Charts status
[[Helm]]
`helm status vault`  For heml deployment status (what, **when**, status)
`helm get manifest vault` For helm manifest details

#### #Raft #Storage 
`vault operator raft list-peers`

#### operational #logs
`kubectl logs vault-x | head -n 15` to get fisrt 15 lines of logs from pod vault-x
`kubectl logs vault-0 > vault-0.log`

#### #audit #logs 
- Vault server should have one or more [audit devices](https://www.vaultproject.io/docs/audit) enabled to trace requests and responses.
- To access file based log (note: file may reside at different location)
`kubectl exec vault-0 -- tail -n 1 /vault/audit/vault_audit.log | jq`
- To copy the file
`kubectl cp default/vault-0:/vault/audit/vault_audit.log vault-0_audit.log`

#### #debugging
You can use `vault debug` in a Kubernetes environment with a multi-step process that involves entering the pod, changing into a writable directory, and executing the command. You can then exit the pod, and copy the resulting debug archive from the pod to the host.

1. Access the pod `kubectl exec vault-0 --stdin=true --tty=true -- sh`
2. Change to /tmp `cd /tmp`
3. Run the debug command `vault debug -interval=1m -duration=2m`  (note: ensure a high privilege VAULT_TOKEN token is used to ensure all possible information is collected)
4. exit the pod `exit`
5. copy the file from /tmp to local `kubectl cp default/vault-0:/tmp/vault-debug-2021-01-22T14-51-12Z.tar.gz vault-debug-2021-01-22T14-51-12Z.tar.gz`
6. Extract the file `tar --extract --gunzip --file vault-debug-2021-01-22T14-51-12Z.tar.gz`


#kubernetes #troubleshooting 
