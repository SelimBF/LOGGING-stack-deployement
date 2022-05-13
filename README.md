# LOG
log lover 


deployement of full Monitoring/Loging stack  on K3S cluster

 if you finally realize that ELK is a huge resource consumable Grafanalab has an alternatives that could replace it ,
 using  Grafana-Loki and Fluentd-bit (CNCF project) in this lab we will deploy a whole stack 
 for loging and monitoring (for Prometheus "only" deployment check https://github.com/SelimBF/minikube-monitoring/):
 
 

Requirement :


This lab could be deplyed standalone in a single node but for better performance we will use :
(you could  skip worker deployement)

:::: 2 Centos 8 VM's ::::  

-----------------------------------------------------deploy Master node ------------------------------------------------------- 

 #1- disable firewlld
 
    systemctl disable firewalld --now

#2- Install K3s:

    curl -sfL https://get.k3s.io | sh -
    kubectl get node 
    NAME     STATUS     ROLES                  AGE    VERSION
    server   Ready      control-plane,master   10d    v1.22.7+k3s1
 
 -----------------------------------------------------deploy helm packager manager -------------------------------------------
 
 #3- Install helm:

    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh
    
-----------------------------------------------------deploy Prometheus-operator/garafana -------------------------------------   
 #4- Install Prometheus-operator/garafana
 
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm install prometheus prometheus-community/prometheus
    kubectl edit svc prometheus-server(ClusterIP ==> NodePort/Loadbalancer)

    helm repo add grafana https://grafana.github.io/helm-charts
    helm install grafana grafana/grafana
    kubectl edit svc grafana(ClusterIP ==> NodePort/Loadbalancer)
    helm repo update
   
  #5- changing default grafana password in order to gain access
  
   note: this could be skipped if you dont face passwrd auht issue 

    kubectl exec -it [pod-name] /bin/bash

    bash-5.1$ 
         cd /usr/share/grafana
         grafana-cli admin reset-admin-password 'admin'  (login to grafana using login/passwoord : admin/admin)
 
     use master @ and  port in browser level to test grafana/prometheus are working fine :
      example :http://@master:port/
      
 -----------------------------------------------------deploy Worker node ------------------------------------------------------- 
    
#6 joining a new worker to the cluster

    curl -sfL https://get.k3s.io | K3S_URL=https://<server>:6443 K3S_TOKEN=<token> sh -
 
     K3s Token obtained from : cat /var/lib/rancher/k3s/server/node-token
                                  &
                       ip addr of server (master node)

 note: the worker hostname in the cluster must be different to master hostname if you face such problem just change worker hostname with 
       
       hostnamectl set-hostname XXXX
  
 
  
  
  #7 deploying Loki-stack :(loki-promtail-fluentbit)


     helm repo add loki https://grafana.github.io/loki/charts
     helm repo update

    helm upgrade --install loki loki/loki-stack --set fluent-bit.enabled=true,promtail.enabled=true
    
    kubectl edit svc Prometheus(ClusterIP ==> NodePort/Loadbalancer) wq!
    kubectl edit svc loki(ClusterIP ==> NodePort/Loadbalancer) wq!
    kubectl edit svc grafana(ClusterIP ==> NodePort/Loadbalancer) wq!
    
![image](https://user-images.githubusercontent.com/74049018/168298430-7118bdc3-b1b5-4004-b312-715a808f99bf.png)

now configure grafana data source (loki/prometheus) and  explore loki log using query https://grafana.com/docs/loki/latest/logql/
  

import some Loki/prometheus  dash from here:  https://grafana.com/grafana/dashboards/
![image](https://user-images.githubusercontent.com/74049018/168295970-1fca7dfb-1ae7-4fac-b51b-4cd6cf40e8e8.png)

  in order to assure log reception in loki level we deploy Fluent & Promtail where both ingest log , you could deploy only one of them or use it for gather others log file   
  
  another faced issue is that Promtail/fluentd status changed to LoopBackcrash  instead of Running so we deploy them as docker container outiside of the cluster :
  
  ![image](https://user-images.githubusercontent.com/74049018/168295349-6d366244-4f1e-4bc8-9f80-461950479886.png)

    docker run -v /var/log:/var/log -e LOG_PATH="/var/log/*.log" -e LOKI_URL="http://localhost:3100/loki/api/v1/push" grafana/fluent-bit-plugin-loki:latest
    dokcer ps /podman ps 
    
![image](https://user-images.githubusercontent.com/74049018/168295188-ea5e6482-143f-49ed-b52a-ec8557b33cc9.png)

  
  ENJOY Log analytics now with edge native monitoring and loging stack  
  
  

  
    
