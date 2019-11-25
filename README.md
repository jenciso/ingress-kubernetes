## Ingress install

Install
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
curl -Ls https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml -o service-nodeport.yaml
sed -i "s/type: NodePort/type: LoadBalancer/" service-nodeport.yaml
kubectl apply -f service-nodeport.yaml
```

Check installation

```
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```

## Create a ssl certificate


Create a bundle certificate
```
(cat certificate.crt; printf "\n"; cat ca_bundle.crt) > certificate-bundle.crt
kubectl create secret tls  tls-wildcard--paas --key private.key --cert certificate-bundle.crt -n ingress-nginx
```
Add in the deployment 
```
--default-ssl-certificate=$(POD_NAMESPACE)/tls-wildcard--paas
```

## HA-proxy

Add IP alias

```
ip a add 192.168.56.20/24 dev enp0s8
```
Insert configuration into the haproxy.cfg

```
frontend ingress-80
    bind        192.168.56.20:80
    mode tcp
    default_backend ingress-80
backend ingress-80
    balance roundrobin
    mode tcp
    server ingress 10.0.2.20:80 check

frontend ingress-443
    bind        192.168.56.20:443
    mode tcp
    default_backend ingress-443
backend ingress-443
    balance roundrobin
    mode tcp
    server ingress 10.0.2.20:443 check
```
