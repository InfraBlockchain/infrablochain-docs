---
title: 릴레이 체인 배포
description: 릴레인 체인을 배포하는 방법을 알아봅니다.
keywords:
  - 릴레인 체인
---

## 배포

서버에 chain을 배포 하는 방법에 대해 작성되어 있는 페이지입니다.

설명에 앞서, 이 문서에서는 네트워크를 통해 chain spec file을 다운로드받아 사용하는 방법만을 설명합니다. 만약 이렇게 진행을 원치않을 경우, 사용자께서 원하는 방법을 사용하셔서 chain spec 파일을 등록하시기를 바랍니다.

서버 배포 방법은 3가지 방법이 있습니다.

- systemd
- docker
- kubernetes

3가지 방법을 사용하여 배포하는 방법을 알아보겠습니다. 

### Systemd

Systemd는 Linux 호스트에서 서비스를 관리하는 일반적인 방법입니다. 프로세스가 활성화되어 실행 중인지 확인하고, 재시작에 대한 정책을 설정할 수 있으며, 호스트를 실행하는 사용자를 설정하고, 메모리 사용량을 제한하는 등의 작업을 수행할 수 있습니다.

또한 환경 변수 파일을 사용하여 변수를 서버별 자체 파일로 추상화할 수 있습니다.

Systemd를 사용하기 위해서 먼저 service로 등록을 해주어야 합니다. 그러기 위해서 /etc/systemd/system 경로에 Infrablockspace.service를 만들어줍니다. 만든 다음 아래 코드를 복사 붙여넣기 해준 다음 `<binary file>` 과 `<options>`에 필요한 옵션 값들을 넣어줍니다.
`<binary file>`은 relay chain일때 infrablockspace를 입력해주면 되며, para chain은 infrablockspace-parachain을 입력해줍니다.

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

작성 후, 다음 명령어를 통해 service를 등록해줍니다.

```bash
systemctl enable Infrablockspace
systemctl start Infrablockspace
```

### Docker

docker를 활용할 때에는 docker-compose.yml 파일을 사용해서 실행하는 것을 추천합니다. 

docker compose  실행 전, 먼저 chain spec 파일을 다운로드 받아야 합니다. 아래 명령어를 통해 다운로드 받아줍니다.

릴레이 체인의 경우,

```bash
curl -L <chain spec url> -o chain-spec/raw-local-chainspec.json
```


파라 체인의 경우,

```bash
curl -L <chain spec url> -o chain-spec//tmp/raw-parachain-chainspec.json
```

다운로드 받은 파일은  chain-spec이라는 directory에 넣어줍니다. 없을 경우, 생성해줍니다.

그런 다음 docker compose 관련 파일을 아래와 같이 작성해줍니다.

릴레이 체인의 경우,

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

파라 체인의 경우,

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
파라체인은 relay 체인의 boot node id가 필수적으로 필요합니다. 따라서 <bootnode id>를 채워줘야 합니다.


다음 명령어를 통해 실행해줍니다.

```bash
docker compose up -d 
```

### Kubernetes

kubernetes에서 chain을 배포하기 위해서는 다음과 같은 파일들을 작성해줘야 합니다.

이때 dynamic pvc가 가능하다는 전제입니다. 만약 dynamic pvc가 안될경우 pv 생성을 해주시기를 바랍니다.

relay chain의 경우,

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

para chain에 경우 작성법에 대해 알아보겠습니다.
아래 작성 코드는 infra did에 관련하여 작성했습니다.

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

kubernetes에 아래 명령어를 통해 생성해주면 정상적으로 코드가 생성되는 것을 확인할 수 있습니다.

```bash
kubectl apply -f pvc.yaml -f service.yaml -f statefuleset.yaml
```

helm과 operator에 경우 개발중에 있어 추후에 업데이트 될 예정입니다.