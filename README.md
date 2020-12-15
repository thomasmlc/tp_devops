# TP_DevOps_Final

## 1. Namespace

### Decoupage

Je pense qu'un découpage en trois partie est le plus logique.
Un namespace pour le front, un pour le back et un pour le monitoring.
```
kubectl get namespaces

NAME                   STATUS   AGE
default                Active   2d6h
kube-node-lease        Active   2d6h
kube-public            Active   2d6h
kube-system            Active   2d6h
monitoring             Active   27h
namespace-back         Active   2d5h
namespace-front        Active   2d5h
```
### namespace-front

Le front contient le wordpress.
```
NAME                        READY   STATUS    RESTARTS   AGE
wordpress-bfbfdcd5f-5vdkz   1/1     Running   0          2d5h
wordpress-mariadb-0         1/1     Running   0          2d5h
```

### namespace-Back
Le back contient la partie KubeDb.
```
NAME                               READY   STATUS    RESTARTS   AGE
kubedb-operator-644b675d5b-4hjtg   1/1     Running   21         22h
myadmin-66cc8d4c77-j9ksg           1/1     Running   0          22h
myadmin-66cc8d4c77-rhv4r           1/1     Running   0          22h
mysql-wordpress-0                  1/1     Running   0          21h
service-mysql-8487598df8-n7clh     1/1     Running   0          22h
service-mysql-8487598df8-vhwjf     1/1     Running   0          22h
```

### monitoring
Ce namespace contient le prometheus et le grafana.

```
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-prometheuss-prometheus-ope-alertmanager-0   2/2     Running   0          27h
grafanaa-7c75ff7554-pdl6x                                1/1     Running   0          27h
prometheus-prometheuss-prometheus-ope-prometheus-0       3/3     Running   1          27h
prometheuss-grafana-6798cc6884-brn7p                     2/2     Running   0          27h
prometheuss-kube-state-metrics-7785c6cc65-q589s          1/1     Running   0          27h
prometheuss-prometheus-node-exporter-4vxff               0/1     Pending   0          27h
prometheuss-prometheus-node-exporter-ctst2               0/1     Pending   0          27h
prometheuss-prometheus-ope-operator-cb9dbff7-9869d       2/2     Running   0          27h
```

## 2.Kubedb

### Mise en place

Nous avons tout d'abord utilisé Chocolatey pour installer HELM qui est un gestionnaire de paquets pour Kubernetes.  
```
choco install kubernetes-helm
```

On ajoute ensuite Kubedb a notre repo helm afin de permettre son installation dans n'importe quel namespace.  
```
helm repo add appscode https://charts.appscode.com/stable/
helm repo update
```
Afin de verifier son installation on peut utiliser
```
helm repo list

NAME                    URL
bitnami                 https://charts.bitnami.com/bitnami
--> appscode                https://charts.appscode.com/stable/
prometheus-community    https://prometheus-community.github.io/helm-charts
stable                  https://charts.helm.sh/stable
grafana                 https://grafana.github.io/helm-charts
```
Nous pouvons désormais l'installer sur le namespace de notre choix qui est donc namespace-back.

```
helm install kubedb-operator appscode/kubedb -n namespace-back
helm install kubedb-catalog appscode/kubedb-catalog -n namespace-back
```
Le kubedb-catalog nous permettra de mettre en place des pods mysql.

nous devons désormais créer un service mysql et un pod mysql.  
[service-mysql.yaml](service-mysql.yaml)
```
kubectl create -f service-mysql.yaml -n back 
```
Afin de récuperer l'IP de notre phpmyadmin on fait
```
kubectl get service -n namespace-back
```
![ip_phpadmin](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_1.jpg)

Nous devons également créer une bdd mysql avec un pod.  
[docker-mysql.yaml](docker-mysql.yaml)
```
kubectl create -f docker-mysql.yaml -n back
```
Grace à l'IP récupérée plus haut on peut accéder a notre phpmyadmin.  
On doit récuperer l'ip de notre serveur avec cette commande.
```
kubectl get pods mysql-wordpress-0 -n namespace-back -o yaml 
```
![ipserveurmysql](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_2.jpg)

Afin de récuperer notre username et notre password nous devons fouiller les secrets de notre namespace et notament mysql-wordpress-auth.
```
kubectl get secrets mysql-wordpress-auth -n namespace-back -o yaml
```
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_3.jpg)

Une fois decodé ca nous donne 
```
username: root
password: 7WMLM&p)ws-,jx2c
```
C'est ici que la partie kubedb s'arrete car après ca une erreur est apparue nous empechant de nous connecter une fois sur deux et lorsque nous arrivon a nous connecter nous sommes rammené sur la page de login après quelques manipulations.  ![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_4.jpg)
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_5.jpg)  

### CRD

Ou custom ressource comme son nom l'indisque permet de créer de nouvelles ressources et donc ici nous permettre de créer la ressource MYSQL.

## 3. Wordpress

### Installation

Nous utilisons encor HELM mais sur un namespace différent
```
helm repo add bitnami https://charts.bitnami.com/bitnami -n front
helm install wordpress bitnami/wordpress -n namespace-front
```
Ainsi avec la commande 
```
kubectl get service -n namespace-front

NAME                TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE
wordpress           LoadBalancer   10.0.51.171   40.88.243.233   80:32581/TCP,443:30938/TCP   2d6h
wordpress-mariadb   ClusterIP      10.0.144.27   <none>          3306/TCP                     2d6h
```
Nous récupérons l'IP externe qui permet d' acceder à notre wordpress.
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_7.jpg)
Nous pouvons également récuperer les identifiant administrateur avec les secrets.  
```
kubectl get secrets wordpress -n namespace-front -o yaml
````
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_6.jpg)  
Ce qui donne 
```
kDcPpQ0VEV
```
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_8.jpg)

## 4. RBAC 
On peut créer les roles dans azure a l'aide du Access control
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_13.jpg)
malheuresement n'ayant rien de vraiment fonctionnel mais roles sont faux et inutiles.  
Les actions que l'on peut autoriser ou interdir sont listés [ici](https://docs.microsoft.com/fr-fr/azure/role-based-access-control/resource-provider-operations)

## 5. Monitoring
On installe prometheus et grafana sur nnotre namespace monitoring grace a HELM. 
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

helm install prometheuss prometheus-community/prometheus -n monitoring

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install grafanaa grafana/grafana -n monitoring
```
On peut ensuite les lancer 
```
kubectl port-forward -n monitoring prometheus-prometheuss-prometheus-ope-prometheus-0 9090
 
kubectl port-forward -n monitoring grafanaa-7c75ff7554-pdl6x 3000
```
On peut ainsi se connecter sur grafana sur notre localhost:3000  
Pour trouver les mots de passe il faut fouiller les secrets.

```
kubectl get secrets grafanaa -n monitoring -o yaml
```
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_9.jpg)

ce qui donne
```
user: admin
password: leFCroOZ70N4MtaspXklNDx7R9Lc2wXKzLz0dwsF
```
![](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_10.jpg)

En allant dans les data-source on devrait retrouver notre prometheus ou alors nous le créons nous meme.  
![ipserveurmysql](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_11.jpg)

en ainsi en important un dashboard de base avec l'ID 315 nous obtenons
![ipserveurmysql](https://github.com/thomasmlc/tp_devops/blob/main/images/Screenshot_12.jpg)
