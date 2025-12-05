# Kind
## Criar Cluster
```bash
kind create cluster --config cluster.yaml --name meucluster
```

### Listar o Cluster
```bash
kind get clusters
```

### Deletar um Cluster
```bash
kind delete cluster --name meucluster
# or
kind delete cluster --config cluster.yaml
```