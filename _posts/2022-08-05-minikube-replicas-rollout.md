---
layout: post
title: minikube + kube + replicas + rollout + ConfigMap
date: 2022-08-05
tags: wsl
---

minikube 關於 replicas 與 rollout 測試

我是用 minikube 自建 registry
```
minikube addons enable registry
docker run --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:$(minikube ip):5000" &
```
git 下來 IBM 範例
```
git clone https://github.com/ibm-developer-skills-network/CC201.git
cd CC201/labs/3_K8sScaleAndUpdate/
```

把內容換 minikube 的 registry (因裡面有 / 換成 ? 去取代指令的 /)
```
sed -i 's?us.icr.io/<my_namespace>?localhost:5000?g' deployment.yaml
```

確認 cat deployment.yaml |grep localhost:5000
```
image: localhost:5000/hello-world:1
```

docker image 放 minikube 的 registry
```
docker build -t localhost:5000/hello-world:1 . && docker push localhost:5000/hello-world:1
```
確認 image
```
docker images |grep localhost:5000/hello-world
```
deployment image
```
kubectl apply -f deployment.yaml
```

看部屬狀況
```
kubectl get pods
```
發佈 hello-world
```
kubectl expose deployment/hello-world
```
把minikube 推出到 8080 (有開過就不必了)
```
kubectl proxy --port=8080 &
```

```
curl -L http://127.0.0.1:8080/api/v1/namespaces/default/services/hello-world/proxy
```

出現像是
```
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
```

設定 3 個 POD
```
kubectl scale deployment hello-world --replicas=3
```
看部屬狀況
```
kubectl get pods
```

測試看是否有多個 POD
```
for i in `seq 10`; do curl -L http://127.0.0.1:8080/api/v1/namespaces/default/services/hello-world/proxy; done
```
像是出現 其中POD 序號不同（三種）
```
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
Hello world from hello-world-7c88966754-ndwm6! Your app is up and running!
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
Hello world from hello-world-7c88966754-6gg4c! Your app is up and running!
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
Hello world from hello-world-7c88966754-ndwm6! Your app is up and running!
Hello world from hello-world-7c88966754-xzpxb! Your app is up and running!
```
切回1個POD
```
kubectl scale deployment hello-world --replicas=1
```

換字 Hello world from 變成 Welcome to
```
sed -i 's/Hello world from/Welcome to/g' app.js
```

建立第二版
```
docker build -t localhost:5000/hello-world:2 . && docker push localhost:5000/hello-world:2
```

部屬第二版
```
kubectl set image deployment/hello-world hello-world=localhost:5000/hello-world:2
```

看更新的狀態
```
kubectl rollout status deployment/hello-world
```

瀏覽看看是不是第二版 字 Welcome to 
```
curl -L http://127.0.0.1:8080/api/v1/namespaces/default/services/hello-world/proxy
```
出現像是
```
Welcome to hello-world-5c5d6588f-lm5th! Your app is up and running!
```
回上一版
```
kubectl rollout undo deployment/hello-world
```
等成功回版後在 curl 檢查

建立  ConfigMap 顯示 MESSAGE內容
```
kubectl create configmap app-config --from-literal=MESSAGE="This message came from a ConfigMap!"
```

把內容換 deployment-configmap-env-var.yaml 的 換成 minikube registry (因裡面有 / 換成 ? 去取代指令的 /)
```
sed -i 's?us.icr.io/<my_namespace>?localhost:5000?g' deployment-configmap-env-var.yaml
```

看 app.js 的 res.send
```
cat app.js |grep res.send
```

把  res.send換成 configmap-env
```
export var1="res.send('Welcome to ' + hostname + '! Your app is up and running!"
export var2="res.send(process.env.MESSAGE + '"
sed -i "s#$var1#$var2#g" app.js
```

建立image 與部屬
```
docker build -t localhost:5000/hello-world:3 . && docker push localhost:5000/hello-world:3
docker images |grep localhost:5000/hello-world
kubectl apply -f deployment-configmap-env-var.yaml
```

```
curl -L http://127.0.0.1:8080/api/v1/namespaces/default/services/hello-world/proxy
```

看到
```
This message came from a ConfigMap!
```

可換 ConfigMap 內容
```
kubectl delete configmap app-config
kubectl create configmap app-config --from-literal=MESSAGE="This message test !"
```
砍掉原有的 pod 會換成新的
```
kubectl delete pod $(kubectl get pod |grep hello-world|awk '{print $1}')
```

瀏覽
```
curl -L http://127.0.0.1:8080/api/v1/namespaces/default/services/hello-world/proxy
```

顯示
```
This message test !
```
