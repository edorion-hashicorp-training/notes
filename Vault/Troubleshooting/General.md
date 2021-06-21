# General Troubleshooting
https://learn.hashicorp.com/tutorials/vault/troubleshooting-vault?in=vault/operations

#### Jornal 
journalctl -b --no-pager -u vault
journalctl -b | awk '$5 ~ "vault"'

- To dump to file
journalctl -b --no-pager -u vault | gzip -9 > /tmp/"$(hostname)-$(date +%Y-%m-%dT%H-%M-%SZ)-vault.log.gz"

note: verify log local in /etc/systemd/system/vault.service

#### Docker 
docker logs vault0
docker logs vault0 2>&1 | gzip -9 - > vault0.log.gz

#### Kubernetes
kubectl logs vault-55bcb779b4-8mfn6

#### Server gave HTTP response to HTTPS client
The CLI always attempts to use a TLS enabled connection to the server so if server is TLS disabled it will happen... alot

#### Policy testing
- attach [[policy]] to adhock token
vault token create -policy=webapp
- then check [[policy]] capabilites against pth
vault token capabilities s.IcTMGNOug5Cx3wBqpGvI5X4e transit/decrypt/phone-number

#### Vault Debug
- command honors the same variables that the base command accepts

```Bash
vault debug -interval=1m -duration=10m 
tar xvfz vault-debug-2019-11-06T01-26-54Z.tar.gz
```

#troubleshooting
