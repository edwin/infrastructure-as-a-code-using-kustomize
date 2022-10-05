# Infrastructure as a code by using Kustomize

## Concept
add a base yaml template as references, and will be patch by `Kustomize` depends on the target environment. It would generate a new `name` and `namespace` depending on the targetted environment.

sample command
```
$ kustomize build overlays/sit
```

will result in something like this
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: example-php-docker-helloworld
  name: sit-example-php-docker-helloworld
  namespace: sit-helloworld-ns
spec:
  ports:
  - port: 8080
  selector:
    app: example-php-docker-helloworld
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: example-php-docker-helloworld
  name: sit-example-php-docker-helloworld
  namespace: sit-helloworld-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example-php-docker-helloworld
  template:
    metadata:
      labels:
        app: example-php-docker-helloworld
    spec:
      containers:
      - env:
        - name: MEM_TOTAL_MB
          valueFrom:
            resourceFieldRef:
              resource: limits.memory
        image: appuio/example-php-docker-helloworld
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 3
        name: example-php-docker-helloworld
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          limits:
            memory: 512Mi
          requests:
            memory: 512Mi
```

if we want to deploy for production, change the build parameter into `prd`
```
$ kustomize build overlays/prd
```

result will be different, compared to when build against `sit`. As we can see, it use another namespace which is `prd-helloworld-ns`
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: example-php-docker-helloworld
  name: prd-example-php-docker-helloworld
  namespace: prd-helloworld-ns
spec:
  ports:
  - port: 8080
  selector:
    app: example-php-docker-helloworld
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: example-php-docker-helloworld
  name: prd-example-php-docker-helloworld
  namespace: prd-helloworld-ns
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-php-docker-helloworld
  template:
    metadata:
      labels:
        app: example-php-docker-helloworld
    spec:
      containers:
      - env:
        - name: MEM_TOTAL_MB
          valueFrom:
            resourceFieldRef:
              resource: limits.memory
        image: appuio/example-php-docker-helloworld
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 3
        name: example-php-docker-helloworld
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          limits:
            memory: 1250Mi
          requests:
            memory: 1250Mi
```