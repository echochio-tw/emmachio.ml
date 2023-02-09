---
layout: post
title: ingress 重啟 shell
date: 2023-02-09
tags: k8s
---

修改一下shell 讓 ingress 重啟
```
cat <<EOF > ./ingress_restart.sh
#!/usr/bin/bash
mkdir -p ./tmp
_item_list=$(kubectl get ingress -n major|grep -v NAME | awk '{print $1}' )
for _item in ${_item_list}; do
    _file_name=${_item}.yaml
    kubectl -n major get ingress ${_item} -o yaml > ./tmp/${_file_name}
done;
cd ./tmp
kubectl delete -f .
kubectl apply -f .
cd ..
rm -rf ./tmp
EOF
```

修改成可執行
```
chmod +x ./ingress_restart.sh
```

執行就重啟nginx 了
```
./ingress_restart.sh
```
