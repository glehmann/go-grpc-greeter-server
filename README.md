An example gRPC server ready to use to test nginx GRPC and GRPCS backend protocol.

# Usage

This is the exact example proposed in https://kubernetes.github.io/ingress-nginx/examples/grpc/ , just ready to use.

The image is available on docker hub: https://hub.docker.com/r/glehmann/go-grpc-greeter-server .

~~~
docker pull glehmann/go-grpc-greeter-server
~~~

On a local k8s server like kind or k3d, `kubectl apply -f` this configuration

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: go-grpc-greeter-server
  name: go-grpc-greeter-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-grpc-greeter-server
  template:
    metadata:
      labels:
        app: go-grpc-greeter-server
    spec:
      containers:
      - image: glehmann/go-grpc-greeter-server
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 50m
            memory: 50Mi
        name: go-grpc-greeter-server
        ports:
        - containerPort: 50051

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: go-grpc-greeter-server
  name: go-grpc-greeter-server
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 50051
  selector:
    app: go-grpc-greeter-server
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
  name: fortune-ingress
  namespace: default
spec:
  rules:
  - host: grpctest.localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-grpc-greeter-server
            port:
              number: 80
  tls:
  # This secret must exist beforehand
  # The cert must also contain the subj-name grpctest.dev.mydomain.com
  # https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/PREREQUISITES.md#tls-certificates
  - secretName: ca-key-pair
    hosts:
      - grpctest.localhost
~~~

The service is available on https://grpctest.localhost

With `grpcurl`, `grpcurl grpctest.localhost:443 list`.

# Building the image

~~~
docker build -t glehmann/go-grpc-greeter-server .
docker push glehmann/go-grpc-greeter-server
~~~
