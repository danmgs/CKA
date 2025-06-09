# CKA


## Chemins pratiques

| Topic             | Chemin |
| ----------------- | --------------------------------------------------- |
| apt k8s           | cat /etc/apt/sources.list.d/kubernetes.list
| cni               | /etc/cni/net.d/<10-canal.conflist> 
| static pods k8s   | /etc/kubernetes/manifests


<ins>kubelet<ins>:

```
systemctl status kubelet

controlplane:~$ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

```

=> ***/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf***

=> ***/etc/kubernetes/kubelet.conf***

=> ***/var/lib/kubelet/config.yaml***  (mentionne le chemin "staticPodPath": ***/etc/kubernetes/manifests*** )





## Commandes système utiles

- sudo netstat -natulp | grep postgres | grep LISTEN

- systemctl list-units --type=service --all

- openssl x509 -in <certificat>  -text -noout

- openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text

- sed -i 's/blue/red/g' fichier.txt

- ps -ef | grep --color=auto etcd

## Commandes kubectl utiles


### Divers

- kubectl run httpd --image=httpd:alpine --port=80 --expose

Exposer un pod avec un service. Peut etre necessaire de renommer le service par la suite (--dry-run=client -o yaml)

- kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-

- kubectl get node node01 --show-labels

- kubectl label node node01 color=blue

- kubectl label node node01 color-

- kubectl get po elephant -o yaml > elephant.yaml

- kubectl replace -f elephant.yaml --force.

This command will delete the existing one first and recreate a new one from the YAML file.

- watch kubectl get powatch kubectl top pod

- kubectl run static-busybox --image=busybox --dry-run=client --command sleep 1000 -o yaml

- k get hpa --watch

- kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123

echo -n "password" | base64
Toujours utiliser -n pour encoder des secrets en Base64 car supprime le dernier caractère de saut de ligne !



### Custom columns

```
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
```

### Events

kubectl get events -> tous les events lié pod/service..etc dans le namespace actif

kubectl get events -n kube-system   -> events lié à un namespace donné e.g "kube-system" ici

```
kubectl get event --field-selector involvedObject.name=<pod-name>kubectl get events --sort-by='.metadata.creationTimestamp' -A

kubectl get events -o yaml -> explique pourquoi on peut utiliser "involvedObject"

kubectl get events -o yaml | grep -i involve -C2
```

### Tri


```
k get pod -n spectra-1267  -o custom-columns="POD_NAME:.metadata.name,IP_ADDR:.status.podIP" --sort-by=".status.podIP"  > /root/pod_ips_cka05_svcn

k get pod -n spectra-1267  -o custom-columns="POD_NAME:.metadata.name,IP_ADDR:.status.podIP" --no-headers | sort -k2  > /root/pod_ips_cka05_svcn
```

"sort -k2 -r" => -r pour tri décroissant

### Execution de pod container

```
k exec -it webapp-pod -- /bin/bash

k exec -it webapp-pod -- env
```

### Creation d'un pod temporaire avec exécution de commande (puis suppression du pod)


```
k run podtmp -it --image=busybox:1.28 --restart=Never --rm -- /bin/sh

k run podtmp -it --image=busybox:1.28 --restart=Never --rm -- <cmdhere>
```

https://stackoverflow.com/questions/44712874/how-do-i-run-a-container-from-the-command-line-in-kubernetes-like-docker-run


## Troubleshooting divers

- Inspect the kube-apiserver PodCheck the status of the kube-apiserver static pod:
```
crictl ps | grep kube-apiserver
```

Get the logs:crictl logs <container_id>

- kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=node01

- Check to see if I can do everything in my current namespace ("*" means all), with verbose mode

```
kubectl auth can-i --as=system:serviceaccount:ci:ci-sa --namespace=production '*' '*' --v=10
```


## Configurations diverses

### /etc/sysct.conf

To ensure that IP forwarding changes are persistent:Create a configuration file: vi /etc/sysctl.d/k8s.confAdd the following line to the file: net.ipv4.ip_forward=1Apply the changes: sysctl -p

plus simple
vi /etc/sysct.conf-> decommenter la ligne "net.ipv4.ip_forward=1"


## Kubernetes Network Policies:

- **{}** in podSelector means select all pods within the namespace.

- **[]** typically isn't used directly in podSelector but indicates no matches when used within label expressions.apiVersion: networking.k8s.io/v1

```
kind: NetworkPolicy
metadata:
  name: backend-external-access-np
  namespace: dev
spec:
  egress:
  - {}
  podSelector: {}
  policyTypes:
  - Egress
```

Allow all ingress
```
spec:
  egress:
  - {}
  podSelector: {}
  policyTypes:
  - Egress  
```
 

Allow nothing
```
spec:
  egress: []
  policyTypes:
  - Egress
```

Allow nothing  (same bis)
```
spec:
  policyTypes:
  - Egress
```

## DNS


### Adresse d'un pod est : <P-O-D-I-P.default.pod>

> kubectl get pod nginx-resolver -o wide
> kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod

Get the IP of the nginx-resolver pod and replace the dots(.) with hyphon(-) which will be used below.

> kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup 172-17-1-11.default.pod
If you don't see a command prompt, try pressing enter.
Address 1: 172.20.0.10 kube-dns.kube-system.svc.cluster.local

Name:      172-17-1-11.default.pod
Address 1: 172.17.1.11 172-17-1-11.nginx-resolver-service.default.svc.cluster.local
pod "test-nslookup" deleted
