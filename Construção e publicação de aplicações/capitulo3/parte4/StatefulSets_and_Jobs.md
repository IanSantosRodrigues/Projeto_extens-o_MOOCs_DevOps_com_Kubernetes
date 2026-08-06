# StatefulSets, Volumes Persistentes e Jobs

## O que você aprenderá nesta página
- Persistir dados com PV, PVC e StorageClass
- Entender StatefulSets e Headless Services
- Executar tarefas finitas com Jobs

Até aqui, os Pods podiam ser substituídos sem grande preocupação. Isso funciona bem para aplicações sem estado, mas não para bancos de dados, filas, caches persistentes e serviços que precisam de identidade estável. Nesses casos, precisamos combinar armazenamento persistente, `StatefulSet` e, em alguns cenários, `Job`.

## Volumes persistentes

`PersistentVolume` (PV) representa uma unidade de armazenamento disponível no cluster. `PersistentVolumeClaim` (PVC) é o pedido feito pela aplicação. `StorageClass` descreve como o armazenamento pode ser provisionado dinamicamente.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Um Pod ou Deployment pode montar esse PVC:

```yaml
volumeMounts:
  - name: data
    mountPath: /usr/share/nginx/html
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data
```

## StatefulSet

`StatefulSet` é o controlador indicado quando a aplicação precisa de identidade estável. Os Pods são criados com nomes previsíveis, como `postgres-0`, `postgres-1` e `postgres-2`. Um StatefulSet normalmente trabalha com um Headless Service para oferecer nomes DNS individuais para cada Pod.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

## Jobs

`Job` executa uma tarefa finita até a conclusão. Ele é útil para migrações, cargas iniciais, scripts administrativos e processamentos pontuais. Use Deployment para aplicações sem estado, StatefulSet quando identidade e volume por réplica forem importantes, e Job para tarefas que terminam.
