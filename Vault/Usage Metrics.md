https://learn.hashicorp.com/tutorials/vault/usage-metrics?in=vault/operations

- difficult to scope enterprise
-  Vault 1.6 introduced Vault Metrics dashboard in Web UI... basically shows client count / usage stats without having to go through telemetry monitoring
-  enabled by default for ENT but not OSS
-  to enable for OSS
	-  `vault write sys/internal/counters/config enabled=enable`


- accessed vi the UI via:
	- "status"
		- "metrics"

-   **Active clients** is the primary metric on which pricing is based. It is the sum of unique entities (or distinct entities) and active direct tokens (or non-entity tokens).
    
-   **Unique entities** (distinct entities) are representations of a particular user, client, or application. These can be created manually or automatically as a result of previously-unknown users authenticating to Vault. These entities have had an active login to Vault within the one-year period. (If you are unfamiliar with Vault entities, refer to the [Identity: Entities and Groups](https://learn.hashicorp.com/tutorials/vault/identity) tutorial.)
    
-   **Active direct tokens** ([non-entity tokens](https://www.vaultproject.io/docs/concepts/client-count#understanding-non-entity-tokens)) are tokens issued without an entity attached. This is because some customers or workflows might avoid using entity-creating authentication methods and instead depend on token creation through the Vault API.


- Data displayed is for the current #namespaces 
- Data is only recorded from the time the feature is enabled (hence why always for ENT version as it relates to billing)
- Data is collected (process'd / tagged / kv'd ) in monthly blocks not daily

**Caution:** When data collection is disabled in the middle of a month, all data for that month will be deleted.

### Vault auditor tool
[Vault-auditor binaries](https://releases.hashicorp.com/vault-auditor/)
The tool uses the audit device logs to count usage history
**NOTE:** `vault-auditor` currently supports only audit device logs from Vault **versions 1.3** through **1.5**.

### KMIP client metrics
For KMIP specific metrics:
`vault-auditor parse -kmip-info audit_log`
