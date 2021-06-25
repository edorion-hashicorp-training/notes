https://www.vaultproject.io/docs/concepts/ha
# HA

- can tell if backend supported by starting backend and seeing if it says it's available
- `ha_storage` (manages the HA storage lock) and `storage` can be separate
- 

### Server-to-Server Communication
- HA state data stored in Vault storage, no over network, just shred storage read by multi clients 
- TLS1.2 certs stored and retrieved from shared store for request forwarding comms
- **client redirection** 307 redirects to active, only needs active node deets from shared storage
- **Request Forwarding** The request is encrypted by shared storage certs and communicated between nodes rather than a redirect. May help if active is not accessible to the requester
- 