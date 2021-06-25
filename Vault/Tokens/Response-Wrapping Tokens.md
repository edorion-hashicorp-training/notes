https://www.vaultproject.io/docs/concepts/response-wrapping

- a wrapped response DOESN'T include the actual secret data
- it contains a token to allow retrieval of the secret data from the single-use token
- resulting in the end point being able to communicate back over TLS (chances are the initial request WOULDN'T have been secure / only HTTP) to receive the secret securely

- creation is by X-Vault-Wrap-TTL header and a tick duration or -wrap-ttl in CLI


Validation is essential to prevent man in the middle atacks where tokens or Vault addr's may be injected by:
1. delay in token receipt
2. verify the token
3. verify the path  

When setting a TTL https://www.vaultproject.io/docs/concepts/policies#required-response-wrapping-ttls