---
title: Relay Chain Deployment
description: Learn how to deploy a relay chain.
keywords:
  - para chain
  - relay chain
  - chain spec
  - infra blockchain
---

## Deployment

This page provides instructions on how to deploy a chain to a server.

Before we begin, please note that this document only covers the method of downloading and using the chain spec file via the network. If you prefer not to proceed this way, you can upload the file or directly create the chain spec file on the server using various methods.

There are three methods for server deployment:

- systemd
- docker
- kubernetes

Let's explore how to deploy using these three methods.

### Systemd

Systemd is a common method for managing services on Linux hosts. It allows you to check if a process is active and running, set policies for restarts, configure the user running the host, and limit memory usage, among other tasks.

You can also use environment variable files to abstract variables into separate files for each server.

To use systemd, you need to first register the service. To do this, create a file named `Infrablockspace.service` in the `/etc/systemd/system` directory. Then, copy and paste the code below, replacing `<binary file>` and `<options>` with the necessary option values. For a relay chain, use `infrablockspace` as the `<binary file>`, and for a para chain, use `infrablockspace-parachain`.

```bash
#/etc/systemd/system/Infrablockspace.service
[Unit]
Description=Infrablockspace Validator

[Service]
User=Infrablockspace
ExecStart=/home/Infrablockspace/<binary file> <options>
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

After writing the file, register the service using the following commands:

```bash
systemctl enable Infrablockspace
systemctl start Infrablockspace
```

### Docker

When using Docker, it is recommended to use a `docker-compose.yml` file for execution.

Before running Docker Compose, you need to download the chain spec file. Use the following command to download the file:

For a relay chain:

```bash
curl -L <chain spec url> -o chain-spec/raw-local-chainspec.json
```

For a para chain:

```bash
curl -L <chain spec url> -o chain-spec//tmp/raw-parachain-chainspec.json
```

Place the downloaded file in the `chain-spec` folder. If the folder does not exist, create it.

Next, create the Docker Compose-related files as shown below:

For a relay chain:

```bash
version: "3.1"
services:
  infrablockspace:
    image: public.ecr.aws/v8x3j0k5/infrablockspace:3
    command:
      [
        "--alice",
        "--validator",
        "--base-path",
        "/data/infrablockspace",
        "--chain",
        "/tmp/raw-local-chainspec.json",
        "--rpc-port=9933", # json rpc
        "--ws-port=9944", #p2p
        "--unsafe-rpc-external",
        "--unsafe-ws-external",
        "--rpc-cors",
        "all"
      ]
    restart: always
    volumes:
      - ./chain-spec:/tmp
      - ./data/infrablockspace:/data/infrablockspace
    ports:
      - "30333"
```

For a para chain:

```bash
version: "3.1"
services:
  infrablockspace:
    image: public.ecr.aws/v8x3j0k5/infra-did-chain:2
    command:
      [
                "--alice",
              "--collator",
              "--base-path",
              "/data/did",
              "--chain",
              "/tmp/raw-parachain-chainspec.json",
              "--rpc-port=9933", # json rpc
              "--ws-port=9944", #p2p
              "--port=30333", #p2p
              "--prometheus-external",
              "--prometheus-port=9615",
              "--unsafe-rpc-external",
              "--unsafe-ws-external",
              "--rpc-cors",
              "all",
              "--",
              "--execution",
              "wasm",
              "--rpc-port=9934",
              "--ws-port=9945", #p2p
              "--port=30334",
              "--chain",
              "/tmp/raw-local-chainspec.json",
              "--base-path",
              "/data/relay",
              "--unsafe-rpc-external",
              "--unsafe-ws-external",
              "--bootnodes",
                <bootnode id>
      ]
    restart: always
    volumes:
      - ./chain-spec:/tmp
      - ./data/infrablockspace:/data/infrablockspace
    ports:
      - "30333"
```

For a para chain, the boot node ID of the relay chain is required. Therefore, you need to fill in `<bootnode id>`.

Run the following command to execute the Docker Compose:

```bash
docker compose up -d 
```

### Kubernetes

To deploy a chain in Kubernetes, you need to create the following files:

Please note that dynamic persistent volume claims are assumed to be possible. If dynamic persistent volume claims are not possible, please create persistent volumes.

For a relay chain:

statefuleset.yaml

```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: infrablockspace-boot
spec:
  # pod selector
  selector:
    matchLabels:
      app: infrablockspace # has to match .spec.template.metadata.labels
      region: an2
      mode: relay-boot
  # select headless service
  serviceName: "infrablockspace-headless-service"
  # update strategy
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
  # spec
  replicas: 1 # by default is 1
  template:
    metadata:
      labels:
        app: infrablockspace # has to match .spec.template.metadata.labels
        region: an2
        mode: relay-boot
    spec:
      terminationGracePeriodSeconds: 10
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 1000
      initContainers:
        - name: download-spec
          image: curlimages/curl:8.1.2
          command: [
              "curl",
              "-L",
              <relay chain spec url>,
              "-o",
              "/tmp/raw-local-chainspec.json",
            ]
          volumeMounts:
            - name: infrablockspace-spec
              mountPath: /tmp
      containers:
        - name: infrablockspace
          image: public.ecr.aws/v8x3j0k5/infrablockspace:3
          imagePullPolicy: Always
          args: [
              "--alice",
              "--validator",
              "--base-path",
              "/data/infrablockspace",
              "--chain",
              "/tmp/raw-local-chainspec.json",
              "--rpc-port=9933", # json rpc
              "--ws-port=9944", #p2p
              "--prometheus-external",
              "--prometheus-port=9615",
              "--unsafe-rpc-external",
              "--unsafe-ws-external",
              "--rpc-cors",
              "all",
              "--ws-max-connections=16000"
            ]
          resources:
            limits:
              cpu: 1000m
              memory: 2Gi
          ports:
            - containerPort: 9933
              name: infraspace-http
            - containerPort: 30333
              name: infraspace-p2p
          volumeMounts:
            - name: infrablockspace-pvc
              mountPath: /data/infrablockspace
            - name: infrablockspace-spec
              mountPath: /tmp
      # volume claim
      volumes:
        - name: infrablockspace-spec
          emptyDir: {}
        - name: chain-keystore
          emptyDir: {}
        - name: infrablockspace-pvc
          persistentVolumeClaim:
            claimName: infrablockspace-pvc
```

service.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: infrablockspace-service-clusterip
spec:
  selector:
    app: infrablockspace
  type: ClusterIP
  ports:
    - name: infraspace-http
      protocol: TCP
      port: 9933
      targetPort: 9933
      # If you set the `spec.type` field to `NodePort` and you want a specific port number,
      # you can specify a value in the `spec.ports[*].nodePort` field.
    - name: infraspace-ws
      protocol: TCP
      port: 9944
      targetPort: 9944
    - name: infraspace-p2p
      protocol: TCP
      port: 30333
      targetPort: 30333
---
apiVersion: v1
kind: Service
metadata:
  name: infrablockspace-headless-service
spec:
  clusterIP: None
  selector:
    app: infrablockspace
  ports:
    - name: infraspace-http
      protocol: TCP
      port: 9933
      targetPort: 9933
    - name: infraspace-ws
      protocol: TCP
      port: 9944
      targetPort: 9944
    - name: infraspace-p2p
      protocol: TCP
      port: 30333
      targetPort: 30333
---
apiVersion: v1
kind: Service
metadata:
  name: infrablockspace-boot-peer
spec:
  selector:
    app: infrablockspace
    mode: relay-boot
  ports:
    - name: infraspace-p2p
      protocol: TCP
      port: 30333
      targetPort: 30333
```

pvc.yaml

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: infrablockspace-pvc
  labels:
    app: infrablockspace-pvc
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
```

For a para chain, let's take a look at the writing method.

The following code is written for Infra DID.

statefuleset.yaml

```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: infra-did
spec:
  # pod selector
  selector:
    matchLabels:
      app: infra-did
      mode: para
  # select headless service
  serviceName: "infra-did-headless-service"
  # update strategy
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
  # spec
  replicas: 1 # by default is 1
  template:
    metadata:
      labels:
        app: infra-did # has to match .spec.selector.matchLabels
        mode: para
    spec:
      terminationGracePeriodSeconds: 10
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 1000
      initContainers:
        - name: download-relay-spec
          image: curlimages/curl:8.1.2
          command: [
              "curl",
              "-L",
              <relay chain spec url>,
              "-o",
              "/tmp/raw-local-chainspec.json",
            ]
          volumeMounts:
            - name: chain-spec
              mountPath: /tmp
        - name: download-did-spec
          image: curlimages/curl:8.1.2
          command: [
              "curl",
              "-L",
              <para chain spec url>,
              "-o",
              "/tmp/raw-parachain-chainspec.json",
            ]
          volumeMounts:
            - name: chain-spec
              mountPath: /tmp
      containers:
        - name: infra-did-chain
          image: public.ecr.aws/v8x3j0k5/infra-did-chain:2
          imagePullPolicy: Always
          args: [
              "--alice",
              "--collator",
              "--base-path",
              "/data/did",
              "--chain",
              "/tmp/raw-parachain-chainspec.json",
              "--rpc-port=9933", # json rpc
              "--ws-port=9944", #p2p
              "--port=30333", #p2p
              "--prometheus-external",
              "--prometheus-port=9615",
              "--unsafe-rpc-external",
              "--unsafe-ws-external",
              "--rpc-cors",
              "all",
              "--",
              "--execution",
              "wasm",
              "--rpc-port=9934",
              "--ws-port=9945", #p2p
              "--port=30334",
              "--chain",
              "/tmp/raw-local-chainspec.json",
              "--base-path",
              "/data/relay",
              "--unsafe-rpc-external",
              "--unsafe-ws-external",
              "--bootnodes",
                <relay boot node id>
            ]
          ports:
            - containerPort: 9933
              name: did-http
            - containerPort: 30333
              name: relay-peer
            - containerPort: 9944
              name: did-ws
          volumeMounts:
            - name: chain-spec
              mountPath: /tmp
            - name: chain-keystore
              mountPath: /keystore
            - name: infra-did-pvc
              mountPath: /data/did
            - name: infra-did-relay-pvc
              mountPath: /data/relay
      # volume claim
      volumes:
        - name: chain-spec
          emptyDir: {}
        - name: chain-keystore
          emptyDir: {}
        - name: infra-did-pvc
          persistentVolumeClaim:
            claimName: infra-did-pvc
        - name: infra-did-relay-pvc
          persistentVolumeClaim:
            claimName: infra-did-relay-pvc
        - name: aura
          secret:
            secretName: infra-did-aura
            defaultMode: 0400
```



service.yaml
```yml
apiVersion: v1
kind: Service
metadata:
  name: infra-did-service-clusterip
spec:
  selector:
    app: infra-did
    mode: para
  type: ClusterIP
  ports:
    - name: infra-did-http
      protocol: TCP
      port: 9933
      targetPort: 9933
    - name: infra-did-ws
      protocol: TCP
      port: 9944
      targetPort: 9944
      # If you set the `spec.type` field to `NodePort` and you want a specific port number,
    #   # you can specify a value in the `spec.ports[*].nodePort` field.
    - name: relay-peer
      protocol: TCP
      port: 30333
      targetPort: 30333
---
apiVersion: v1
kind: Service
metadata:
  name: infra-did-headless-service
spec:
  clusterIP: None
  selector:
    app: infra-did
    mode: para
  ports:
    - name: infra-did-http
      protocol: TCP
      port: 9933
      targetPort: 9933
    - name: infra-did-ws
      protocol: TCP
      port: 9944
      targetPort: 9944
      # If you set the `spec.type` field to `NodePort` and you want a specific port number,
      # you can specify a value in the `spec.ports[*].nodePort` field.
    - name: relay-peer
      protocol: TCP
      port: 30333
      targetPort: 30333
```

pvc.yaml
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: newnal-an2-b-pvc
  labels:
    app: newnal-an2-b-pvc
spec:
  storageClassName: gp3
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: newnal-an2-b-relay-pvc
  labels:
    app: newnal-an2-b-relay-pvc
spec:
  storageClassName: gp3
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
```

By running the following command in Kubernetes, you can confirm that the code is generated correctly.

```bash
kubectl apply -f pvc.yaml -f service.yaml -f statefuleset.yaml
```

For Helm charts and operators, they will be updated in the future during development.

## Learn More

- [Guide: Customize a Chain Specification](/reference/how-to-guides/basics/customize-a-chain-specification/)
- [Node Template Chain Spec](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/node/src/chain_spec.rs)
- [Infra Blockchain Space](/infrablockchain/learn/architecture/infra-blockspace)
- [Parachains](/reference/how-to-guides/parachains/)