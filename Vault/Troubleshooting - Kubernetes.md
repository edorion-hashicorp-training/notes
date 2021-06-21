# Troubleshooting - Kubernetes

#### get vault pods (nodes)
```kubectl get pods```

#### get single node status
- using name from 'kubectl get pods'
```kubectl exec vault-0 -- vault status```

#### multi node status
```for i in {0..2} ; do kubectl exec vault-$i -- vault status ; done```




#kubernetes #troubleshooting 
