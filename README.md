## Ingress install

Install
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
kubectl apply -f service-nodeport.yaml
```

* Create a ssl certificate

```
(cat certificate.crt; printf "\n"; cat ca_bundle.crt) > certificate-bundle.crt
kubectl create secret tls  tls-wildcard--paas --key private.key --cert certificate-bundle.crt -n ingress-nginx
```

* Add in the deployment 

```
kubectl edit deployments -n ingress-nginx nginx-ingress-controller
...

        - --default-ssl-certificate=$(POD_NAMESPACE)/tls-wildcard--paas
```

## HA-proxy


Insert configuration into the haproxy.cfg

```
frontend ingress-80
    bind        192.168.56.10:80
    mode tcp
    default_backend ingress-80
backend ingress-80
    balance roundrobin
    mode tcp
    server ingress 10.0.2.10:30080 check

frontend ingress-443
    bind        192.168.56.10:443
    mode tcp
    default_backend ingress-443
backend ingress-443
    balance roundrobin
    mode tcp
    server ingress 10.0.2.10:30443 check
```


Testing

```
kubectl create deployment nginx-demo --image=nginx
kubectl expose deployment nginx-demo --port 80 --name=nginx-demo
kubectl apply -f ingress-nginx-demo.yml
```
