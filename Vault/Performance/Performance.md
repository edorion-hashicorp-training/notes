# #performance
https://learn.hashicorp.com/tutorials/vault/performance-tuning?in=vault/operations

-   **Linux OS tuning** covers critical OS configuration items for ideal operations.
-   **Vault tuning** details the configuration tuning for Vault itself and primarily includes core configuration items.
-   **Storage backend tuning** contains items of note which are specific to the storage backend in use.

#### #USE
 [USE checklist for Linux](http://www.brendangregg.com/USEmethod/use-linux.html)
-   **Utilization** \- did you get an alert about low storage capacity or notice out of memory errors, for example?
-   **Saturation** \- are there signs that the storage IOPS are at their allowed maximum, for example?
-   **Errors** \- are there errors in the application logs or Vault logs, for example? Are they persistent while performance degrades?

### Linux OS tuning
Verify ulimits (system user constraints) 
`cat /proc/$(pidof vault)/limits`

*Max Open files*
max limit
`cat /proc/sys/fs/file-max`
current limit (cant exceed max)
`cat /proc/$(pidof vault)/limits | awk 'NR==1; /Max open files/'`
For a more detailed view - lsof:
`sudo lsof -p $(pidof vault)`
Can see things specifically like connections to a DB secrets engine

*Errors*
"too many open files"

*Increase the limit*
- [ ] Edit the Vault systemd service unit file, specifying a value for **LimitNOFILE** [process property](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Process%20Properties)
`sudo vi /etc/systemd/system/vault.service`
- [ ] add LimitNOFILE=65536 under the '\[Service\]' propperty then restart the vault demon `systemctl "demon" reload`
- [ ] if no error, restart the vault service `systemctl restart vault` 
- [ ] verify change success `cat /proc/$(pidof vault)/limits | awk 'NR==1; /Max open files/'`
**note:** demon restarts can seal the vault and require unsealing... pain in the @#$%@#%@ if there is no auto unseal

#### #cpu scaling
**note:** Can i/o block as CPU scales in some instances so diminishing returns to a halt

### Vault tuning

#### Cache size
- uses [Least Recently Used (LRU)](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_.28LRU.29)
- tuned with [cache\_size](https://www.vaultproject.io/docs/configuration#cache_size)
- default 131072

#### Max request duration
Two paramaters that can be tuned
- process wide - vault config -  [default\_max\_request\_duration](https://www.vaultproject.io/docs/configuration#default_max_request_duration) default - 90
- listener specific - [max\_request\_duration](https://www.vaultproject.io/docs/configuration/listener/tcp#max_request_duration) - overrides the above default

#### Max request size
- [max\_request\_size](https://www.vaultproject.io/docs/configuration/listener/tcp#max_request_size) - default 32mb

#### HTTP timeouts
4 available configs
- [http\_idle\_timeout](https://www.vaultproject.io/docs/configuration/listener/tcp#http_idle_timeout) - default 5min
- [http\_read\_header\_timeout](https://www.vaultproject.io/docs/configuration/listener/tcp#http_read_header_timeout) - default 10s
- [http\_read\_timeout](https://www.vaultproject.io/docs/configuration/listener/tcp#http_read_timeout) - default 30s
- [http\_write\_timeout](https://www.vaultproject.io/docs/configuration/listener/tcp#http_write_timeout) - default 0s

#### Lease expiration and TTL
All dynamic secrets and tokens have [leases](https://www.vaultproject.io/docs/concepts/lease) - default 32 days

- Short TTL's are good (the shorter the better where possible)
	- for security (shorter attack vector)
	- for performance (less leases less open files)
	- [vault.expire.num\_leases metric](https://www.vaultproject.io/docs/internals/telemetry#policy-and-token-metrics) - gives # leases to be expired
	- can view then in sotrage at either [Inspecting Data in Consul Storage](https://learn.hashicorp.com/tutorials/vault/inspecting-data-consul) or [Inspecting Data in Integrated Storage](https://learn.hashicorp.com/tutorials/vault/inspecting-data-integrated-storage)

### Replication

configured in conf replication stanza
```json
replication {
  resolver_discover_servers = true
  logshipper_buffer_length  = 1000
  logshipper_buffer_size    = "5gb"
}
```

- [`resolver_discover_servers`](https://learn.hashicorp.com/tutorials/vault/performance-tuning?in=vault/operations#resolver_discover_servers)
- [`logshipper_buffer_length`](https://learn.hashicorp.com/tutorials/vault/performance-tuning?in=vault/operations#logshipper_buffer_length
	- entity not byte length
- [`logshipper_buffer_size`](https://learn.hashicorp.com/tutorials/vault/performance-tuning?in=vault/operations#logshipper_buffer_size) - default 10% of total mem or 1GB if inaccessible

- note: verify mem is released if #namespaces etc are deleted

*To improve performance*
- Scale with memory then max configuration to `logshipper\_buffer\_size`

*note:* The old #redis write issue of write frequency impacting i/o etc vs data loss

### #pki #certificates
[PKI Secrets Engine](https://www.vaultproject.io/docs/secrets/pki)
- linear scaling
- as always try to keep TTL's as short as possible
- limit certificate overlap i.e. reuse until renewed rather than have multiple long lived certs
- Certificate Revocation List (CRL) - only 512KB in consul storage so multiple live certs can fill it quick

### #ALC in #policies

- The more complex the more potential processing overhead required to process a request
- policy templates and + and * operators contribute to complexity

###### Policy Evaluation
![Policy Evaluation|800](https://learn.hashicorp.com/img/vault-policy-evaluation.png)


### [[Sentinel]] policies

- understandably the more complex the more expensive it will be 
- template paths add more cost
- Sentinel requests are more expensive than a normal globed policy
- [HTTP import](https://docs.hashicorp.com/sentinel/imports/http/) is a compounding factor in #sentinal policy performance

### #tokens 

Interaction | Vault operations
------------ | ------------
Login request | Write new token to the Token Store<br> Write new lease to the Lease Store
Revoke token (or token expiration) | Delete token<br> Delete token lease<br> Delete all child tokens and leases

[Batch tokens](https://www.vaultproject.io/docs/concepts/tokens#batch-tokens) don't require disk storage so less I/O impact but much less functionality / persistence
- cant be revoked or renewed
- TTL must be set in advance (often longer lived than service tokens)

### Seal Wrap
- #HSM - communication to service adds latency cost
- even if data is cached it will be encrypted so will need to be unencrypted so the cost still exists

### #Storage backend
[storage backends](https://www.vaultproject.io/docs/configuration/storage)
- primary source of latency
- writes cost more than reads 
- The majority of Vault write operations relate to these events:
	-   Logins and token creation
	-   Dynamic secret creation
	-   Renewals
	-   Revocations

###### #consul
- in memory so i/o operations quick
- subject to network latency
- complex 
	- subject to oom 
	- hard to debug
	- snapshotting can impact performance (lock?)
- resource expensive (requires memory of 3x the key/value store containing Vault data)
- imperative to maintain good IOPS ref [Consul reference architecture](https://learn.hashicorp.com/consul/datacenter-deploy/reference-architecture) [Consul deployment guide](https://learn.hashicorp.com/consul/datacenter-deploy/deployment-guide)

**Common errors**
Consul server memory usage and errors
`grep 'Out of memory' /var/log/messages`

Check for "canceled context" which may represent reduced IOPS 
`grep 'ERROR' /var/log/messages`

**Consul storage for vault #tuning**
[kv\_max\_value\_size](https://www.consul.io/docs/agent/options#kv_max_value_size) 
- default 512kb 
- maximum size of data Vault can write to a Consul key
- can revieve "request body too large" errors

txn_max_req_ln
- max length of a transaction body

max_parallel
- default 128
- max parallel / synchronous requests
- more commonly lowered to reduce load on a cluster than increased to go faster

consistency_mode
[Consistency Modes](https://www.consul.io/api-docs/features/consistency)
- again redis like, has trade off's default should be fine unless stale data is of critical importance

###### #Raft 
- simple operation (on disk)
- no network latency but does fsync()
- slower than consul as, disk I/O

**Raft storage tuning**
mlock()
- recommended to be disabled if using raft 
	- issues with BOltDB memory mapped files
	- will load the complete Vault dataset into memory

Swap
- recommended disabled to stop secrets being dumped to disk

#performance_multiplier
- default 0 (behaves like a 5 which isnt the highest performance )
- recommended 1 for production to ensure rapid convergence

#snapshot_threshold
[snapshot\_threshold](https://www.vaultproject.io/docs/configuration/storage/raft#snapshot_threshold)
- like redis writes

### Resource limits & maximums
**Secrets engines**
- no hard limit on number of enabled
- still restricted or max consul value size 51k2b 
	- Raft doesn't impose such a limit but large keys do introduce issues with replication and cluster governance
