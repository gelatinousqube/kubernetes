# kubernetes


create kubedash token

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md


kubectl get secret cicd-token --output=yaml -> pode dar jeito para ver outros yamls
kubectl get secret cicd-token -n cicd --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode

sacar password do grafana do user admin
 kubectl get secret --namespace prometheus prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


copiar um secret de um namespace para outro
 kubectl get secret regcred -n sendit-multichannel -o yaml

Depois basta alterar o namespace e dar apply


systemctl restart kubelet.service
htop para ver recursos

tail -v /var/log/syslog - comando para prender uma pod antes que ela crash - ajuda em troubleshoot

kubeadm config images list


limpar pv que ficam agarrados - tb resulta para pvc
 kubectl patch pv {PVC_NAME} -p '{"metadata":{"finalizers":null}}'
 
show labels
 kubectl get nodes --show-labels
 
add labels on nodes
 kubectl label nodes <node-name> <label-key>=<label-value>



criar um deployment num ns especifico
 kubectl create deployment nginx --image=nginx --namespace=daniel
 
get pod cidr
 kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
 kubectl cluster-info dump | grep -m 1 cluster-cidr

definida cidr

 /etc/kubernetes/manifests/kube-controller-manager.yaml
 

kubectl get events - ver ultimos eventos do cluster - ajudar em troubleshoot

get pod shell
 kubectl exec -it <POD_NAME> -n <NAMESPACE> -- /bin/bash


scale up/down
 kubectl scale deployment/connect --replicas=X -n <NAMESPACE>
 
editar deployments no k8s
 kubectl edit pod <POD_NAME> -n <NAMESPACE>
 
 
kubectl cp <file-spec-src> <file-spec-dest>

kubectl -n <NAMESPACE> cp $LPORTAL_LIFERAY_POD:/opt/liferay/data .


kubectl cp zeebe-hazelcast-exporter.jar sendit-multichannel/zeebe-66d769848d-wqghh:/usr/local/zeebe/exporters

Port forwarding de uma pod 
 kubectl -n prometheus port-forward prometheus-grafana-685496b9b-lxss9 3000 

Port forwading de um serviço
 kubectl port-forward svc/defend-es 9200:9200 -n defend-test
 
kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces

ver os recursos que todas as pods estã a consumir, filtradas por memoria
kubectl top pods -A --sort-by=memory

ver recursos a ser consumidos pelo cpu
 kubectl top pods -A

vai procurar por todas as dentro do namespace defend, faz grep por NodeAffinity, 
apanha tudo da primeira coluna, passa para o kubectl e apaga as pods
kubectl get pods -n defend | grep NodeAffinity | awk '{print $1}' | xargs kubectl delete pod -n defend

kubectl get pods -n ama-site | awk '{print $1}' | xargs kubectl delete pod -n ama-site

links uteis
  https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/
  https://kubernetes.io/pt/docs/reference/kubectl/cheatsheet/
  https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/  
 networking 
  https://blog.neuvector.com/article/advanced-kubernetes-networking
  https://blog.neuvector.com/article/kubernetes-networking
 node selectors
  https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
  https://faun.pub/node-labels-and-selectors-in-kubernetes-76fbd8a8940c
   
apanhar secret para ter novo token e fazer novo join
 kubeadm token create --print-join-command
 
copiar .kube e meter a cena a rodar noutra maquina
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
awk 'FNR==1 {print "---"}{print}' kubernetes/*.yaml | envsubst > kub-app.yaml

listar nodes com taint

 kubectl get nodes -o json | jq '.items[].spec.taints'
 kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints --no-headers 
 
 Retirar taint do node master
  kubectl taint nodes kubernetes node-role.kubernetes.io/master:NoSchedule
  
  
  prender uma pod
  no container, alinhado com as env ou image acrescentar o seguinte
  - commando
    - tail
    - -f
    - /dev/null
    
 assim dá tempo para a pod ser criada e fica presa para andar a mexer lá dentro
 
atenção que ao alterar, tenho de colocar '-' atrás de env


deploy nginx for test
 kubectl create deployment nginx --image=nginx --namespace=<NS>



acrescentar nodeSelector em /etc/kubernetes/manifests/kube-apiserver.yaml
exemplo de como deve ficar

 annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.1.106:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.1.106
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction,PodNodeSelector
#acrescentou-se PodNodeSelector
Assim ocnsigo usar o node-selector para selecionar em que pod é lançado um namespace
------------------------------------------------------------------------------------

kubectl get nodes --show-labels
elastic-search-748f5f7ffc-6j84z
adicionar labels
kubectl label node <NODE> <LABEL>=<VAL>

todas as pods que sejam deployed no namespace, só vão ser publicadas em nós com aquela label

Isto faz parte do deployment
exemplo de namespace 'trancado' para a label: env=leinad

      containers:
        - name: httpd2-server
          image: httpd
          imagePullPolicy: Always
          ports:
            - containerPort: 80
            - containerPort: 443
      nodeSelector:
        env: leinad
-------------------------------------

kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces
kubectl get pod -o wide -A
 
kubectl top pods --sort-by=cpu -A
kubectl top pods --sort-by=memory -A
kubectl describe nodes reapernode2
kubectl get pods -n defend | grep NodeAffinity | awk '{print $1}' | xargs kubectl delete pod -n defend
kubectl top pods -A --sort-by=memory
