# Deployment 매니페스트 작성

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
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - edgehb-web
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
        image: docker.io/chulbori/nginx:20210507
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
      serviceaccountName: heesoo
