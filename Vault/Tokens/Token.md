# Tokens
https://www.vaultproject.io/docs/concepts/tokens

- Managed by / the token store is a backend that manages all the tokens [`token` authentication backend](https://www.vaultproject.io/docs/auth/token)


Lease, Renew, Revoke
- On every auth, a token is leased
	- each token has lease metadata containing
		- ttl
		- if renewable
	- ensures regular checking for tokens and more details audit timeline (which is why all dynamic secrets require a lease)
	- 
	
- [[Policies]] are mapped on authentication

##### #Accessor
1.  Look up a token's properties (not including the actual token ID)
2.  Look up a token's capabilities on a path
3.  Renew the token
4.  Revoke the token

### TTL
value of 0 = never expire
system max TTL = 32 days but cant be extended
mount specific TTLS's overwrite system TTLS's

Explicit max TTL's = hard limit on token life

### Periodic tokens

Have a max ttl and will continue to exist as long as they are renewed. Basically as long as a system can heart beat a token they can keep it

Can be created by:
1.  By having `sudo` capability or a `root` token with the `auth/token/create` endpoint
2.  By using token store roles
3.  By using an auth method that supports issuing these, such as AppRole

### CIDR bound tokens
- white listed token usability


### Service Token
Service tokens are what users will generally think of as "normal" Vault tokens. They support all features, such as renewal, revocation, creating child tokens, and more. They are correspondingly heavyweight to create and track.

### Batch Token
Batch tokens are encrypted blobs that carry enough information for them to be used for Vault actions, but they require no storage on disk to track them. As a result they are extremely lightweight and scalable, but lack most of the flexibility and features of service tokens.



#tokens 