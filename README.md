# AI-for-K8S
AI for K8s SRE

### Local AI Setup
This is free, Open Source OpenAI alternative.

## Download Docker & docker-compose
```
apt install -y docker
snap install docker
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o docker-compose
chmod +x docker-compose
mv docker-compose /usr/local/bin/
```

## Clone Local AI
```
git clone https://github.com/go-skynet/LocalAI
cd LocalAI
```

## Download gpt4all-j to models/
```
wget https://gpt4all.io/models/ggml-gpt4all-j.bin -O models/ggml-gpt4all-j
docker-compose up -d --pull always
```

## Check API is accessible at localhost:8080

```
curl http://localhost:8080/v1/models

curl http://localhost:8080/v1/completions -H "Content-Type: application/json" -d '{
     "model": "ggml-gpt4all-j.bin",            
     "prompt": "How are you Jonny!!",
     "temperature": 0.7
   }'
```

### Setup K8s Cluster


# Download k8sgpt
```
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.21/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb
```

# Add LocalAI 
```k8sgpt auth add -p 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```

# Add OpenAI 
```
#k8sgpt auth add --backend localai --model <model_name> --baseurl http://localhost:8080/v1 --temperature 0.7
k8sgpt auth add --backend localai --model <model_name> --baseurl http://localhost:8080/v1
```

### # If errors comes execute below

```rm -rfv ~/.config/k8sgpt/k8sgpt.yaml```

# Create sample yaml

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

## Deply yaml & delete PV

```kubectl -f apply sample.yaml; kubectl delete pv task-pv-volume```

## Test

```k8sgpt analyze --explain --backend localai```


