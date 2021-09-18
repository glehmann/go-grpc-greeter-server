An example gRPC server ready to use to test nginx GRPC and GRPCS backend protocol.

# Usage

This is the exact example proposed in https://kubernetes.github.io/ingress-nginx/examples/grpc/ , just ready to use.

# Building the image

~~~
docker build -t glehmann/go-grpc-greeter-server .
docker push go-grpc-greeter-server
~~~
