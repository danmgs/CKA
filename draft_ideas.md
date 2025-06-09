

root@controlplane:~# ssh node01
root@node01:~# ps -ef |  grep /usr/bin/kubelet
root        4147       1  0 14:05 ?        00:00:00 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
root        4773    4733  0 14:05 pts/0    00:00:00 grep /usr/bin/kubelet

root@node01:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/just-to-mess-with-you



The kube-api server stopped again! Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue.
Run crictl ps -a command to identify the kube-api server container. Run crictl logs container-id command to view the logs.
watch -n 1  crictl ps -a


kubectl auth whoamikubectl auth can-i get pods --as=jane.doe --namespace=my-namespace

kubectl set serviceaccount deploy/web-dashboard dashboard-sa
k rollout restart deployment <deploymentname>  relance un deploiement (et kill les pods pour en refaire d'autres)



---

  quelle est la commande kubectl permettant de savoir si un service est bien attaché à un pod (sans voir less definitions du service et du pod et les selectors et labels),

kubectl get endpoints

kubectl get ep <service-name> -o yaml  

---
  Trouver à quels Services un Pod est rattaché est super utile pour du troubleshooting !  
kubectl get svc --selector app=backend,env=prod
---detail d'une revision (voir les details et son image, avec l'option --revision)kubectl rollout history deployment video-app --revision=3

---creer un job "toto " (cka-cronjob-manual) a partir du cronjob cronjob/cka-cronjob
kubectl create job --from=cronjob/cka-cronjob cka-cronjob-manual
--
voir la spec des resources k8sk explain <resources> --recursive  

permet aussi de voir le group auquel appartiel la resources , utile pour construire les rules pour les roles et clusterroles =>    apiGroups: ["batch"]

rules:- apiGroups: [""] # # at the HTTP level, the name of the resource for accessing Pod # objects is "pods" resources: ["pods"] verbs: ["get", "list", "watch"]- apiGroups: ["batch"] # # at the HTTP level, the name of the resource for accessing Job # objects is "jobs" resources: ["jobs"] verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
si pas de group, alors apiGroups: [""]  (pour les pods et svc par exemple)

  apiGroups: ["apps"]   pour les deploiements---

etcd externe

chown -R etcd:etcd /var/lib/etcd 
car etcd installé comme un service

cat  /etc/systemd/system/etcd.service

contient
