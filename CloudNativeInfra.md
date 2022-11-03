### Deployment 매니페스트

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: edgehb-web
  namespace: edgehb-homepage
  labels:
    app: web
    service: heesoo
    env: prod
  annotation: 
    env: dev
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: web
      service: heesoo
      env: prod
  template:
    metadata:
      namespace: edgehb-homepage
      labels:
        app: web
        service: heesoo
        env: prod
    spec:
      serviceaccountName: imagedn
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - web
              topologyKey: kubernetes.io/hostname
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: name
                    operator: In
                    values:
                      - worker-node
      containers:
      - name: web
        image: kurlycorp.kr.dev/nginx:latest
        resources:
          requests:
            memory: "300Mi"
            cpu: "200m"
          limits:
            memory: "1Gi"
            cpu: "1"
        ports:
        - containerPort: 8080
          protocol: TCP
          name: http
        livenessProbe:
          initialDelaySeconds: 60
          failureThreshold: 5
          periodSeconds: 10
          httpGet:
            path: /state/version
            port: http
        readinessProbe:
          initialDelaySeconds: 10
          failureThreshold: 3
          httpGet:
            path: /state/version
            port: http
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - sleep 50
```

### 정상동작 확인 방법 및 결과
shell#> kubectl get all,ingress -n {namespace}. 
![resource](https://user-images.githubusercontent.com/117291027/199703560-e40b4af5-5253-428f-9a79-1a7194e3b93c.png)


shell#> curl https://{exam_url} 
![curl](https://user-images.githubusercontent.com/117291027/199703550-9e937b61-43da-467e-82d2-0be64d4fc8ca.png)
