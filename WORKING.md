Hi Gopal,

Thanks for those! I can confirm that a leadership election did happen as shown in the logs below. The cause at a Vault level appears to be that nzp2it3 was unable to contact the rest of the nodes (and conversely the rest of the nodes speaking to it) which speaks to something network related being the root cause which you may want to investigate further in the infrastructure where Vault is being hosted. 

I understand you said Vault is functioning normally, by this you have run `vault operator raft list-peers` and seen all 5 nodes showing up correctly? It appears you have the logs set to `warn` which is great for saving on storage but can make life difficult looking at issues retrospectively so it may be advisable to increase the detail moving forwards as detailed in [Vault Server Logs](https://learn.hashicorp.com/tutorials/vault/troubleshooting-vault#vault-server-logs). As the logs supplied appear to stop around 2021-06-24T16:03:40 is this when the logs get rolled over for the day? If so then there may be additional information i the following days logs which would be interesting to look at, if not then I would expect everything went back to normal (in the absence of more detailed logging). 

```
**nzp2it1**
2021-06-24T16:03:30.352+1200 [WARN]  storage.raft: rejecting vote request since we have a leader: from=nzp2it2v.nz.unix.anz:8201 leader=nzp2it3v.nz.unix.anz:8201
...
...
2021-06-24T16:03:33.096+1200 [WARN]  storage.raft: heartbeat timeout reached, starting election: last-leader=nzp2it3v.nz.unix.anz:8201

**nzp2it2**
2021-06-24T16:03:30.350+1200 [WARN]  storage.raft: heartbeat timeout reached, starting election: last-leader=nzp2it3v.nz.unix.anz:8201

**nzp2it3**
2021-06-24T16:03:24.916+1200 [WARN]  storage.raft: failed to contact: server-id=nzp2it1v.nz.unix.anz time=3.224575146s
2021-06-24T16:03:25.028+1200 [WARN]  storage.raft: failed to contact: server-id=nzp2it2v.nz.unix.anz time=3.277092355s
2021-06-24T16:03:25.029+1200 [WARN]  storage.raft: failed to contact: server-id=nzp2it5v.nz.unix.anz time=3.312642081s
2021-06-24T16:03:25.029+1200 [WARN]  storage.raft: failed to contact: server-id=nzp2it4v.nz.unix.anz time=3.438288171s
2021-06-24T16:03:25.029+1200 [WARN]  storage.raft: failed to contact quorum of nodes, stepping down
2021-06-24T16:03:25.050+1200 [WARN]  core: leadership lost, stopping active operation

**nzp2it5**
2021-06-24T16:03:30.357+1200 [WARN]  storage.raft: rejecting vote request since we have a leader: from=nzp2it2v.nz.unix.anz:8201 leader=nzp2it3v.nz.unix.anz:8201
...
2021-06-24T16:03:33.104+1200 [WARN]  storage.raft: rejecting vote request since we have a leader: from=nzp2it1v.nz.unix.anz:8201 leader=nzp2it3v.nz.unix.anz:8201
...
 2021-06-24T16:03:34.104+1200 [WARN]  storage.raft: heartbeat timeout reached, starting election: last-leader=nzp2it3v.nz.unix.anz:8201
 ```
 
 Regards
 Andrew
 
 
 Hi Javier,
 
 Thanks for reaching out to Vault support. 
 
 It seems your question has lost a bit of context on the way through to us. 
 
 For the avoidance of doubt can you please clarify:
 - what product your referencing with vault\_aws\_secret\_backend? i.e The actual Vault backend or the Teraform provider or something else?
 - additional detail to your use case 

Regards
Andrew
 
 
 
 
 --- for review ---
Hi Andrew,

Before getting to call can we please confirm that the syslog audit device is not working? 

By default Vault will log to the AUTH syslog facility. While it depends on the distribution or image you are using, generally speaking, default installations of rsyslog / syslog will place logs using the AUTH syslog facility into `/var/log/auth.log`. It is possible to change where they are logged either by rsyslog configuration (which is a bit out of our support scope) or by declaring a different facility when enabling the audit device. As an example `vault audit enable syslog tag="vault" facility="daemon"`.  If syslog data is handled different to auth logs then it may be something to consider if you actually want Vault logs exposed in it.

If you don't see any of the vault logs your expecting in /var/log/auth.log so we can do some testing before a call can you please:
- supply the output of `vault audit list -detailed`
- supply the output of `ps -eaf | grep log`
- supply the content or a copy of /var/log/vault/vault.log
- supply what OS and version your running vault on

Regards
Andrew



VAULT_skip_verify on cli cmmand exec

113565
export vault addr to ip address




for rose:
-  need to update 




- kv1 vs v2 and list perms on that path






Hi Raul,

Sorry for the delay in response but thank you for reaching out to Vault support.

Looking at the error your supplied you have 2 error's 

1 - It is saying the vault server doesn't have permissions to create a file in /var/log. 

Depending on your permission configuration it may not be possible to create a file or folder in /var/log unless you are root. As it is not advisable to run Vault as a root user and so to not necessarily expose the content of /var/log it may be better to create a folder within /var/log to store your Vault logs with something like the following:
`sudo mkdir /var/log/vault`
`sudo chown vault:vault /var/log/vault`
Then `vault audit enable file file_path=/var/log/vault/vault_audit.log`  should work. 

Note: The above is a recommendation on production server configuration and does rely on the Vault server being run as the vault user. If the user running the server is different then the folder ownership will need to be changed appropriately. Please let me know if you need more assistance with and if so, supply the output of `ps -eaf | grep vault` and for additional information on audit log opperation, you can reffer to this (article)[https://support.hashicorp.com/hc/en-us/articles/4402469308307-How-to-enable-file-audit-device-in-Vault-Enterprise-setup-]

2 - When you run `sudo vault` you are just changing the user that is running the cli not the user that is running the server that is trying to create the log file. As you are getting a certificate error I expect you have CA details exported in your normal user but those are not available during a sudo process. As sudo execution of the vault cli has no effect on the vault server operation I would suggest steering clear of this method moving forward. However, if you need to access different Vault installations sudoing to another non root user that has the relevant environment variables already setup may be practical. 

Regards
Andrew








Hi Michael,

Thanks for getting back to me. Sorry from your initial comment I was under the impression you were experiencing the issues when using Terraform. For the avoidance of doubt I have been able to test the policy you have supplied with out issues in Vault to create namespaces from root, then in the child from root enable an auth method, create a new user with the same policy and then do the same again from the child to a new child namespace. So the policy you have supplied really should be able to do what you want and if it isn't then i'd expect something else to be going on i.e. another policy explicitly denying access or you have an exported VAULT_NAMESPACE variable that is overriding a command level declaration.

Taking Terraform out of the equation and simply focusing on Vault so i can determine your effective permissions and why you are getting errors can you please supply the following commands for both the root and one other namespace (grouped by namespace):
 - 'vault token lookup'
- 'vault token capabilities /sys/namespaces'
- `for v in $(vault policy list); do echo "#### $v"; vault policy read "$v"; done`
- `echo $VAULT_NAMESPACE`
- `vault namespace list -namespace=<insert name of namespace>` 

Regards
Andrew





--- for review --- 

Hi Teerasak,

Thanks for reaching out to Vault support.

The policy you supplied pertains to a kv v2 secrets engine being used and can produce that kind of error if used against a kv v1 engine. You can verify this by running `vault secrets list -detailed` and looking under the Options column. If it is "version:1" then you can either alter your policy or upgrade the secrets engine to v2. 

The altered policy should look like this

```
# Read-only permission on 'secret/data/myapp/*' path
path "secret/myapp/*" {
  capabilities = [ "read" ]
}
```

To upgrade the engine (which may be risky depending range of use), you can follow the process outlined [here](https://www.vaultproject.io/docs/secrets/kv/kv-v2#upgrading-from-version-1)

Please let me know if you encounter and further issues.

Regards
Andrew