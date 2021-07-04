-  [ ] Teraform / vault / Travis CI / github 
-  [ ] [[Name spaces]] (expand)
-  [ ] https://learn.hashicorp.com/tutorials/vault/inspecting-data-consul?in=vault/monitoring
-  [ ] #certification #exam 
-  [ ] sills map submission
-  [ ] review  [Go template language](https://golang.org/pkg/text/template/)



KB article

- [ ] By default Vault will log to the AUTH syslog facility. While it depends on the distribution or image your using, generally speaking, default installations of rsyslog / syslog will place logs using the AUTH syslog facility into `/var/log/auth.log`. It is possible to change where they are logged either by rsyslog configuration (which is a bit out of our support scope) or by declaring a different facility when enabling the audit device. As an example `vault audit enable syslog tag="vault" facility="daemon"`.  If syslog data is handled different to auth logs then it may be something to consider if you actually want Vault logs exposed in it. 
- [ ] 