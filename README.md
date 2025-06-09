# CKA


## Chemins pratiques

| Topic             | Chemin |
| ----------------- | --------------------------------------------------- |
| apt k8s           | cat /etc/apt/sources.list.d/kubernetes.list
| cni               | /etc/cni/net.d/<10-canal.conflist> 
| config ipforward  | etc/sysctl.conf
| static pods k8s   | /etc/kubernetes/manifests
| Logs pods         | /var/log/pods/
| Logs container    | /var/log/containers/



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

- crictl ps -a

- crictl ps -a | grep -i exited

- crictl logs <containerid>

- nc -zv <host> <port>

- scp test_file.txt remote_server_username@remote_server_IP:/remote/directory

- grep -i ^ERROR    (recherche les éléments commençant par ERROR)

## Commandes kubectl utiles


### Divers

- kubectl run httpd --image=httpd:alpine --port=80 --expose

Exposer un pod avec un service. Peut etre necessaire de renommer le service par la suite (--dry-run=client -o yaml)

- k run nginx-resolver-cka06-svcn --image=nginx --expose=true --port 80 --dry-run -o yaml > amodifier.yaml

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

- k history -h

- k history undo deployment [deployment]

- k history pause deployment [deployment]

on peut voir qu'un déploiement est en pause avec la commande describe

- k history resume deployment [deployment]

- k history status deployment [deployment] --version [version]

- k get pod kube-controller-manager-[xxx]

### Custom columns

```
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
```

### Explain

k explain netpol.spec --recursive

k explain pod.spec.containers --recursive | grep -i volumeMounts -A 20


```
> k explain pod.spec.containers --recursive | grep -i envFrom -A 20


  envFrom       <[]EnvFromSource>
    configMapRef        <ConfigMapEnvSource>
      name      <string>
      optional  <boolean>
    prefix      <string>
    secretRef   <SecretEnvSource>
      name      <string>
      optional  <boolean>
```

k explain pod.spec.containers --recursive |grep -i env -A10

```
controlplane:~/autoscaler/vertical-pod-autoscaler$ k explain pod.spec.containers.env --recursive
KIND:       Pod
VERSION:    v1

FIELD: env <[]EnvVar>


DESCRIPTION:
    List of environment variables to set in the container. Cannot be updated.
    EnvVar represents an environment variable present in a Container.

FIELDS:
  name  <string> -required-
  value <string>
  valueFrom     <EnvVarSource>
    configMapKeyRef     <ConfigMapKeySelector>
      key       <string> -required-
      name      <string>
      optional  <boolean>
    fieldRef    <ObjectFieldSelector>
      apiVersion        <string>
      fieldPath <string> -required-
    resourceFieldRef    <ResourceFieldSelector>
      containerName     <string>
      divisor   <Quantity>
      resource  <string> -required-
    secretKeyRef        <SecretKeySelector>
      key       <string> -required-
      name      <string>
      optional  <boolean>

```

exemple de yaml d'un pod container utilisant un configmap pour une variable d'environnement "APP_COLOR"

    spec:
      containers:
      - name: my-container
        env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-wl10-config-map
              key: APP_COLOR


```
controlplane:~/autoscaler/vertical-pod-autoscaler$ k explain pod.spec.containers.envFrom --recursive
KIND:       Pod
VERSION:    v1

FIELD: envFrom <[]EnvFromSource>


DESCRIPTION:
    List of sources to populate environment variables in the container. The keys
    defined within a source must be a C_IDENTIFIER. All invalid keys will be
    reported as an event when the container is starting. When a key exists in
    multiple sources, the value associated with the last source will take
    precedence. Values defined by an Env with a duplicate key will take
    precedence. Cannot be updated.
    EnvFromSource represents the source of a set of ConfigMaps

FIELDS:
  configMapRef  <ConfigMapEnvSource>
    name        <string>
    optional    <boolean>
  prefix        <string>
  secretRef     <SecretEnvSource>
    name        <string>
    optional    <boolean>
```

exemple de yaml d'un pod container utilisant un configmap pour TOUTES les variables d'environnement du configmap


```
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: myconfigmap
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
- To ensure that IP forwarding changes are persistent:

Create a configuration file: `vi /etc/sysctl.d/k8s.conf`

Add the following line to the file: net.ipv4.ip_forward=1

Apply the changes: `sysctl -p`

- Plus simple
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

Allow all egress
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

Autres Notes:

- Quand le pod accepte un ingress sur le port 80, alors cela signifie implicitement aussi que la communication est également sortante par 80


- **OR condition** on ingress.from.namespaceSelector AND ingress.from.podSelector

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-net-policy
spec:
  egress:
  - ports:
    - port: 80
      protocol: TCP
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
    - podSelector:
        matchLabels:
          app: cyan-white-cka28-trb
    ports:
    - port: 80
      protocol: TCP
  podSelector:
    matchLabels:
      app: cyan-app-cka28-trb
  policyTypes:
  - Ingress
  - Egress

-------

controlplane:~$ k describe netpol my-net-policy
Name:         my-net-policy
Namespace:    default
Created on:   2025-06-09 22:52:05 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=cyan-app-cka28-trb
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      NamespaceSelector: kubernetes.io/metadata.name=default
    From:
      PodSelector: app=cyan-white-cka28-trb
  Allowing egress traffic:
    To Port: 80/TCP
    To: <any> (traffic not restricted by destination)
  Policy Types: Ingress, Egress

```

- **AND condition** on ingress.from.namespaceSelector AND ingress.from.podSelector

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-net-policy
spec:
  egress:
  - ports:
    - port: 80
      protocol: TCP
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
      podSelector:
        matchLabels:
          app: cyan-white-cka28-trb
    ports:
    - port: 80
      protocol: TCP
  podSelector:
    matchLabels:
      app: cyan-app-cka28-trb
  policyTypes:
  - Ingress
  - Egress

---

controlplane:~$ k describe netpol my-net-policy
Name:         my-net-policy
Namespace:    default
Created on:   2025-06-09 22:52:05 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=cyan-app-cka28-trb
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      NamespaceSelector: kubernetes.io/metadata.name=default
      PodSelector: app=cyan-white-cka28-trb
  Allowing egress traffic:
    To Port: 80/TCP
    To: <any> (traffic not restricted by destination)
  Policy Types: Ingress, Egress

```

## DNS


### Adresse d'un service

```
<service>.namespace.svc.local
```

### Adresse d'un pod est : 

```
<podip_avec_des_tirets>.namespace.pod.local
```

<P-O-D-I-P.default.pod>

> kubectl get pod nginx-resolver -o wide
> kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod

Get the IP of the nginx-resolver pod and replace the dots(.) with hyphon(-) which will be used below.

> kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup 172-17-1-11.default.pod
If you don't see a command prompt, try pressing enter.
Address 1: 172.20.0.10 kube-dns.kube-system.svc.cluster.local

Name:      172-17-1-11.default.pod
Address 1: 172.17.1.11 172-17-1-11.nginx-resolver-service.default.svc.cluster.local
pod "test-nslookup" deleted


### Construire un VPA

- Se baser sur un HPA

exemple de HPA ici https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

- Remplacer `apiVersion: autoscaling/v2` par  `apiVersion: autoscaling.k8s.io/v1`

Pour récupérer l'api version d'un vpa, on peut s'aider des infos via `k explain vpa` ou `k api-resources |grep -i vpa`

- Remplacer `spec.scaleTargetRef` par `spec.targetRef`

- Remplir le VPA avec la doc `k explain vpa.spec --recursive`

En general pour `updatePolicy.updateMode` valant Auto |Initial | Off | Recreate

un exemple de VPA

```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
  namespace: services
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: api-container
      minAllowed:
        cpu: "600m"
        memory: "600Mi"
      maxAllowed:
        cpu: "1"
        memory: "1Gi"
```


### Helm

helm lint [path]


### Voir le range IPS des pods et service d'un cluster

Range IPs des pods sur l'ensemble du cluster

```
controlplane:~$ k get -n kube-system pod kube-controller-manager-controlplane -o yaml | grep -i cidr
    - --allocate-node-cidrs=true
    - --cluster-cidr=192.168.0.0/16
```

Range IPs des services

```
controlplane:~$ k get -n kube-system pod kube-controller-manager-controlplane -o yaml | grep -i range
    - --service-cluster-ip-range=10.96.0.0/12
```


Range IPs des pods pour un noeud donné du cluster
On notera que ce range d'IP des Pods pour un noeud est inclu dans le range d'ip des pods du cluster (mentionné précédemment: cluster-cidr=192.168.0.0/16), il est en /24 ci-dessous mais aurait pu rester en /16.

```
k get node controlplane -o jsonpath='{.spec.podCIDR}'

=> 192.168.0.0/24

k get node node01 -o jsonpath='{.spec.podCIDR}'

=> 192.168.1.0/24

```
Autre moyen simple et grossier pour le range IP des pods pour chaque noeud:
```
controlplane:~$ k get node -o yaml | grep -i podCIDR
    podCIDR: 192.168.0.0/24
    podCIDRs:
    podCIDR: 192.168.1.0/24
    podCIDRs:

```
