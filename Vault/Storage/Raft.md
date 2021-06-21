# #Raft
#Integrated #Storage backend

https://www.vaultproject.io/docs/configuration/storage/raft

Data gets replicated as per [Raft Consensus Algorithm](https://raft.github.io/ "The Raft Consensus Algorithm")

-   **High Availability** – the Integrated Storage backend supports high availability.
-   **HashiCorp Supported** – the Integrated Storage backend is officially supported by HashiCorp.

```storage "raft" { path \= "/path/to/raft/data" node\_id \= "raft\_node\_1" } cluster\_addr \= "http://127.0.0.1:8201"```

*cluster_addr* must be supplied when used in a cluster
*ha_storage* cant be used in conjunciton
*disable_mlock to true* to stop clash of disk use and swapping

Each `retry_join` stanza may contain either a `leader_api_addr` value or a cloud `auto_join` configuration value, but not both.