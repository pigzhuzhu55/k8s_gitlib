# k8s_gitlib

安装部署gitlab.yaml

```
  kubectl apply -f gitlib.yaml
```

注意几个细节：

需要更改gitlib所在主机要绑定的域名如 gitlib.xx.com ，在gitlib.yaml和 /config/gitlab.rb  这2个文件

我之前部署的很多基于k8s部署的插件，最终都有用nginx-ingress转发，部署文件在nginx-ingress.yaml。k8s的nginx服务，nginx的80端口绑定并向外暴露23456端口，这里外部网络只要访问这个23456端口就可以,在k8s集群内部，通过nginx80端口转发到每个集群内部的服务。

比如本例

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gitlab-ingress
  namespace: gitlab
  annotations:
      nginx.ingress.kubernetes.io/force-ssl-redirect: "false" # 强制http重定向到https
      nginx.ingress.kubernetes.io/proxy-body-size: "0" # 设置client_max_body_size为0
spec:
  rules:
  - host: gitlib.xx.com  # 你主机要绑定的域名，后面会通过nginx转发
    http:
      paths:
      - path: /
        backend:
          serviceName: gitlab-service
          servicePort: 80 
```

增加一个访问权，这样就可以访问gitlib.xx.com:23456，转发到gitlab-service这个服务上。可能会发现域名后面加个端口号很恶心，那就再做一层转发到80端口，通过外部另一个转发器去代理到23456就可以。