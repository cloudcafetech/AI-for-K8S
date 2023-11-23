# AI-for-K8S
AI for K8s SRE

## Local AI Setup
This is free, Open Source OpenAI alternative.

#### Download docker 
```
apt install -y docker
snap install docker
```

#### Download Models & Image
```
mkdir ai-models
wget https://gpt4all.io/models/ggml-gpt4all-j.bin -O ai-models/ggml-gpt4all-j
docker image pull quay.io/go-skynet/local-ai:latest
```

#### Run Local AI as a container
```docker run -d -p 8080:8080 -e REBUILD=false -v $PWD/ai-models:/models -ti quay.io/go-skynet/local-ai:latest --models-path /models --context-size 700 --threads 4```

## Check API is accessible at localhost:8080

```
curl http://localhost:8080/v1/models

curl http://localhost:8080/v1/completions -H "Content-Type: application/json" -d '{
     "model": "ggml-gpt4all-j",
     "messages": [{"role": "user", "content": "How are you Jonny!!"}],
     "temperature": 0.7
   }'
```

## Setup K8s Cluster
```curl -s https://raw.githubusercontent.com/cloudcafetech/kubesetup/master/host-setup.sh | KUBEMASTER=<MASTER-IP> bash -s master```

#### Download k8sgpt
```
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.21/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb
```

#### Add OpenAI 
```k8sgpt auth add --backend openai -p <PUT OPEN AI KEY>```

#### Add LocalAI 
```k8sgpt auth add --backend localai --model ggml-gpt4all-j --baseurl http://localhost:8080/v1``` 

#### Modify default for AI provider
```vi ~/.config/k8sgpt/k8sgpt.yaml```

### Create sample yaml

```
cat << EOF > sample.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: tnginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
EOF
```

### Deply yaml & delete PV

```kubectl -f apply sample.yaml; kubectl delete pv task-pv-volume```

### Test

```k8sgpt analyze --explain --backend localai```


