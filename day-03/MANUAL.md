## Manifest
```yaml
# deployment specification
apiVersion: apps/v1
kind: Deployment
metadata: 
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: giropops
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  # pod specifications
  template: 
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.16.0
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 256Mi
          requests:
            cpu: 0.3
            memory: 64Mi  
```

## Subir o yaml
```bash
$ kubectl apply -f deployment.yaml
```

## Criar o yaml do deployment / namespace
```bash
kubectl create deployment --image nginx --replicas 3 nginx-deployment --dry-run=client -o yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```
```bash
kubectl create namespace giropops --dry-run=client -o yaml > namespace.yaml
```

ou criar o yaml a partir de um deployment
```bash
kubectl get deployment nginx-deployment -n giropops -o yaml
```

## Criar namespaces na mão
```bash
kubectl create namespace giropops
```

## Visualizar pods com o label
```bash
kubectl get pods -l app=nginx-deployment
```

## Visualizar replica set
```bash
kubectl get replicaset -n giropops

NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-666b4bc98f   10        10        10      12s
```

## Visualizar deployment
```bash
kubectl get deployment -n giropoops
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   10/10   10           10          108s
```

## Entrar no pod
```bash
kubectl exec -ti <nomedopod> -- bash
```

## Deletar deployments
```bash
kubectl delete deployments nginx-deployment
```

## Subir na mao o numero de replicas
```bash
$ kubectl scale deployment nginx-deployment -n giropops --replicas 3
deployment.apps/nginx-deployment scaled

$ kubectl get pods -n giropops
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-568b56c745-7xfsb   1/1     Running   0          3m34s
nginx-deployment-568b56c745-jqtmf   1/1     Running   0          3m32s
nginx-deployment-568b56c745-n7rmv   1/1     Running   0          3m34s
```

## Estrategia de deploy
### RollingUpdate - faz aos poucos de acordo com o parametro maxUnavailable
No yaml do deployment dentro do bloco strategy
```yaml
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      # qtd de pod que pode subir a mais (pode ser em %) 
      maxSurge: 1 
      # qtd que vai ficar indisponivel para atualizar (pode ser em %)
      maxUnavailable: 2 
  template: 
  ...
```
### Recriate - mata todos os pods e atualiza ao mesmo tempo, tem downtime mas útil qdo a aplicacao nao pode rodar com versoes diferentes no momento da atualizacao
```yaml
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: 
    type: Recriate
  template: 
  ...   
```

# Rollout
### Verificar o status do deploy
```bash
$ kubectl rollout status deployment nginx-deployment -n giropops
deployment "nginx-deployment" successfully rolled out
```
Se atualizar algo no manifest aplicar e logo após rodar veremos algo como

```bash
$ kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment configured

$ kubectl rollout status deployment nginx-deployment -n giropops
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```

## Rollback
### Utilizacao do Undo para desfazer uma mudanca
Exemplo, a versao atual do nginx estava 1.17.0 visto no describe do deployment
e após realizar o rollout undo ele voltou para a ultima alteracao que no exemplo foi só alterado essa versao para 1.16.0 
```bash
$ kubectl rollout undo deployment nginx-deployment -n giropops
deployment.apps/nginx-deployment rolled back
```
## History
### visualizar as revisoes criadas 
Pode ser alterado a quantidade max para 10 (por exemplo) no manifest dentro do bloco "spec" com o parametro "revisionHistoryLimit: 10"

```bash
$ kubectl rollout history deployment nginx-deployment -n giropops
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
3         <none>
4         <none>
5         <none>
6         <none>
```
Ou podemos ver uma revisao especifica

```bash
$ kubectl rollout history deployment nginx-deployment -n giropops --revision 5
deployment.apps/nginx-deployment with revision #5
Pod Template:
  Labels:	app=nginx-deployment
	pod-template-hash=5bb946cdf9
  Containers:
   nginx:
    Image:	nginx:1.13.0
    Port:	<none>
    Host Port:	<none>
    Limits:
      cpu:	500m
      memory:	256Mi
    Requests:
      cpu:	300m
      memory:	64Mi
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
  Node-Selectors:	<none>
  Tolerations:	<none>
```
e para realizar o rollback para uma revisao especifica 

```bash
$ kubectl rollout undo deployment nginx-deployment -n giropops --to-revision=4
deployment.apps/nginx-deployment rolled back
```

## Pause, Resume e Restart
### Com o Pause
```bash
$ kubectl rollout pause deployment nginx-deployment -n giropops
deployment.apps/nginx-deployment paused
```
Mesmo se rodar o kubectl -f deployment.yaml
ele não irá atualizar.
Mas quando dar o Resume ele irá atualizar os pods caso tenha algo na "fila"

### Com Resume
```bash
$ kubectl rollout resume deployment nginx-deployment -n giropops
deployment.apps/nginx-deployment resumed 

# note que alguns pods começaram a ser atualizados após o resume
$ kubectl get pods -n giropops
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6f4f7685c5-cwmrs   0/1     ContainerCreating   0          2s
nginx-deployment-6f4f7685c5-d6gxs   0/1     ContainerCreating   0          1s
nginx-deployment-6f4f7685c5-lggr8   1/1     Running             0          2s
nginx-deployment-6f4f7685c5-wblkr   0/1     ContainerCreating   0          2s
nginx-deployment-6f4f7685c5-wdb9k   1/1     Running             0          2s
nginx-deployment-6f4f7685c5-xl99w   0/1     ContainerCreating   0          1s
nginx-deployment-7d89c5876d-c2n2c   1/1     Terminating         0          15m
nginx-deployment-7d89c5876d-fdjmp   1/1     Running             0          15m
nginx-deployment-7d89c5876d-jpfhm   1/1     Running             0          15m
nginx-deployment-7d89c5876d-l5m2w   1/1     Terminating         0          15m
nginx-deployment-7d89c5876d-ptrz6   1/1     Running             0          15m
nginx-deployment-7d89c5876d-r2khg   1/1     Running             0          15m
nginx-deployment-7d89c5876d-s9sf9   1/1     Running             0          15m
```

### Restart
Mata todos os pods e sobe novamente
```bash
$ kubectl rollout restart deployment nginx-deployment -n giropops
deployment.apps/nginx-deployment restarted
```