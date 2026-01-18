---
title: 超初心者が Kubernetes の勉強を始めてみた【実用編】
tags:
  - kubernetes
private: false
updated_at: '2025-12-18T07:10:31+09:00'
id: 6fbbff2fc0696776feab
organization_url_name: null
slide: false
ignorePublish: false
---
## この章でやること

* Next.js を、Kubernetesにデプロイする

## 前提

* Docker が動いている
* kind と kubectl が入っている
* Next.js プロジェクトがある（`npm run dev` が動く）

## 1. Next.js を Docker で動かせるようにする

プロジェクト直下に `Dockerfile` を作ります。

> 今回は、npmで書きます。

### 1. Dockerfile

```Dockerfile
# Use the official Node.js image as the base image
FROM node:latest

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install project dependencies
RUN npm install

# Copy the rest of the project files to the working directory
COPY . .

# Build the Next.js app
RUN npm run build

# Expose the port that the Next.js app will run on
EXPOSE 3000

# Start the Next.js app
CMD ["npm", "run", "start"]
```

## 3. Docker イメージをビルドする

```sh
docker build -t next-site:1.0 .
```

ビルドできたか確認

```sh
docker images
```

## 4. kind にイメージを読み込ませる

kind は作ったイメージを、そのままクラスターに渡せばOKです。

```sh
kind load docker-image next-site:1.0
```

## 5. Kubernetes にデプロイする（Deployment / Service）

`k8s-next.yaml` を作成：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: next-site
spec:
  replicas: 1
  selector:
    matchLabels:
      app: next-site
  template:
    metadata:
      labels:
        app: next-site
    spec:
      containers:
        - name: next
          image: next-site:1.0
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
---
apiVersion: v1
kind: Service
metadata:
  name: next-site
spec:
  type: NodePort
  selector:
    app: next-site
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 30080
```

適用：

```sh
kubectl apply -f k8s-next.yaml
```

起動確認：

```sh
kubectl get pods
kubectl get svc next-site
```

Pod が `Running` になっていれば大丈夫です。

## 6. ブラウザでアクセス

```sh
kubectl port-forward svc/next-site 8081:80 --address 0.0.0.0
```
別のウィンドウで、
```sh
curl http://localhost:30080
```

## 7. 更新する場合

Next.js を修正したら、基本はこの流れ：

1. Docker を再ビルド（タグを変えるのがおすすめ）

```sh
docker build -t next-site:1.1 .
```

2. kind に読み込み

```sh
kind load docker-image next-site:1.1
```

3. Deployment の image を更新

```sh
kubectl set image deployment/next-site next=next-site:1.1
```

## まとめ

Nextjsでサイトを公開することができました！次は実際に本物のKubernetesを触ってみようと思います！
