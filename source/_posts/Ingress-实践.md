---
title: Ingress-实践
comments: false
date: 2018-08-10 17:43:08
categories: kubernetes
tags: kubernetes
img:
---
# Ingress 实践

<!-- TOC -->

- [Ingress 实践](#ingress-%E5%AE%9E%E8%B7%B5)
    - [Kubernetes 暴露服务方式](#kubernetes-%E6%9A%B4%E9%9C%B2%E6%9C%8D%E5%8A%A1%E6%96%B9%E5%BC%8F)
        - [LoadBlancer Service](#loadblancer-service)
        - [NodePort Service](#nodeport-service)
        - [Ingress](#ingress)
    - [Ingress 介绍](#ingress-%E4%BB%8B%E7%BB%8D)
    - [Ingress 部署](#ingress-%E9%83%A8%E7%BD%B2)
        - [default-backend](#default-backend)
        - [Ingress Controller](#ingress-controller)
        - [快速部署](#%E5%BF%AB%E9%80%9F%E9%83%A8%E7%BD%B2)
            - [创建相关资源](#%E5%88%9B%E5%BB%BA%E7%9B%B8%E5%85%B3%E8%B5%84%E6%BA%90)
            - [创建ingress-nginx服务](#%E5%88%9B%E5%BB%BAingress-nginx%E6%9C%8D%E5%8A%A1)
            - [测试](#%E6%B5%8B%E8%AF%95)
    - [配置TLS SSL访问](#%E9%85%8D%E7%BD%AEtls-ssl%E8%AE%BF%E9%97%AE)
        - [创建证书](#%E5%88%9B%E5%BB%BA%E8%AF%81%E4%B9%A6)
        - [创建 Secret](#%E5%88%9B%E5%BB%BA-secret)
        - [快速创建 Secret](#%E5%BF%AB%E9%80%9F%E5%88%9B%E5%BB%BA-secret)
        - [重新部署](#%E9%87%8D%E6%96%B0%E9%83%A8%E7%BD%B2)
    - [参考文档](#%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

<!-- /TOC -->

## Kubernetes 暴露服务方式
到目前为止 Kubernetes 暴露服务的有三种方式，分别为  
- LoadBlancer Service
- NodePort Service
- Ingress

### LoadBlancer Service  
LoadBlancer Service 是 Kubernetes 结合云平台的组件，如国外 GCE、AWS、国内阿里云等等，使用它向使用的底层云平台申请创建负载均衡器来实现，有局限性，对于使用云平台的集群比较方便。

### NodePort Service  
NodePort Service 是通过在节点上暴漏端口，然后通过将端口映射到具体某个服务上来实现服务暴漏，比较直观方便，但是对于集群来说，随着 Service 的不断增加，需要的端口越来越多，很容易出现端口冲突，而且不容易管理。当然对于小规模的集群服务，还是比较不错的。

### Ingress  
Ingress解决的是新的服务加入后，域名和服务的对应问题，基本上是一个ingress的对象，通过yaml进行创建和更新进行加载。  

Ingress使用开源的反向代理负载均衡器来实现对外暴漏服务，比如 Nginx、Apache、Haproxy等。Nginx Ingress 一般有三个组件组成：  

- Nginx 反向代理负载均衡器
- Ingress Controller      
Ingress Controller可以理解为控制器，它通过不断的跟 Kubernetes API 交互，实时获取后端 Service、Pod 等的变化，比如新增、删除等，然后结合 Ingress 定义的规则生成配置，然后动态更新上边的 Nginx 负载均衡器，并刷新使配置生效，来达到服务自动发现的作用，简单来说，Ingress这种变化生成一段Nginx的配置，然后将这个配置通过Kubernetes API写到Nginx的Pod中，然后reload
- Ingress  
Ingress 则是定义规则，通过它定义某个域名的请求过来之后转发到集群中指定的 Service。它可以通过 Yaml 文件定义，可以给一个或多个 Service 定义一个或多个 Ingress 规则。 
 
## Ingress 介绍  
> 官网对 Ingress 的定义为管理对外服务到集群内服务之间规则的集合，通俗点讲就是它定义规则来允许进入集群的请求被转发到集群中对应服务上，从来实现服务暴漏。 Ingress 能把集群内 Service 配置成外网能够访问的 URL，流量负载均衡，终止SSL，提供基于域名访问的虚拟主机等等。  
>   
Ingress解决的是新的服务加入后，域名和服务的对应问题，基本上是一个ingress的对象，通过yaml进行创建和更新进行加载。
可以通过ConfigMap提供的Nginx命令修改参数。

---  
## Ingress 部署
### default-backend
default-backend
生成一个默认的后端，如果遇到解析不到的URL或者发生错误就转发到默认后端
dashboard.yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-weblogic-ingress
  namespace: kube-system
spec:
  rules:
  - host: helloworld.eric
    http:
      paths:
      - path: /console
        backend:
          serviceName: helloworldsvc 
          servicePort: 7001
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 80
```

- host 指虚拟出来的域名，具体地址(Ingress-controller那台Pod所在的主机的地址)应该加入/etc/hosts中,这样所有去helloworld.eric的请求都会发到nginx
- path:/console 匹配后面的应用路径
- servicePort 主要是定义服务的时候的端口，不是NodePort.
- path:/ 匹配后面dashboard应用的路径,以前通过访问master节点8080/ui进入dashboard的，但dashboard其实是部署在minion节点中，实际是通过某个路由语句转发过去而已，dashboard真实路径如下:


### Ingress Controller
> $ kubectl create -f ingress-nginx/examples/deployment/nginx/nginx-ingress-controller.yaml

### 快速部署  

#### 创建相关资源
> $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
     
#### 创建ingress-nginx服务    
> $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml

#### 测试  
> $ kubectl get svc  -o wide -n ingress-nginx  
> $ kubectl create -f hello-world-deployment.yaml  

```
apiVersion: apps/v1beta1
kind: Deployment
metadata: 
name: hello-world-deployment
spec: 
replicas: 1
template: 
    metadata: 
    labels: 
        app: hello-world
    spec: 
    containers: 
        - image: "gokul93/hello-world:latest"
        imagePullPolicy: Always
        name: hello-world-container
        ports: 
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata: 
name: hello-world-svc
spec: 
ports: 
    -  port: 8080
        protocol: TCP
        targetPort: 8080
selector: 
    app: hello-world
type: NodePort
```

> $ kubectl create -f hello-world-ingress.yaml  
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
creationTimestamp: 2018-08-03T02:51:13Z
generation: 2
name: hello-world-ingress
namespace: default
resourceVersion: "3668608"
selfLink: /apis/extensions/v1beta1/namespaces/default/ingresses/hello-world-ingress
uid: 15b9c53b-96c8-11e8-9920-00505683568f
spec:
rules:
- http:
    paths:
    - backend:
        serviceName: hello-world-svc
        servicePort: 8080
        path: /
```
> $ curl k8s-node-ip:32483/hello

## 配置TLS SSL访问
### 创建证书

```
# 生成 CA 自签证书
mkdir cert && cd cert
openssl genrsa -out ca-key.pem 2048
openssl req -x509 -new -nodes -key ca-key.pem -days 10000 -out ca.pem -subj "/CN=kube-ca"

# 编辑 openssl 配置
cp /etc/pki/tls/openssl.cnf .
vim openssl.cnf

# 主要修改如下
[req]
req_extensions = v3_req # 这行默认注释关着的 把注释删掉
# 下面配置是新增的
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = dashboard.mritd.me
DNS.2 = kibana.mritd.me

# 生成证书
openssl genrsa -out ingress-key.pem 2048
openssl req -new -key ingress-key.pem -out ingress.csr -subj "/CN=kube-ingress" -config openssl.cnf
openssl x509 -req -in ingress.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out ingress.pem -days 365 -extensions v3_req -extfile openssl.cnf

```

### 创建 Secret
创建好证书以后，需要将证书内容放到 secret 中，secret 中全部内容需要 base64 编码，然后注意去掉换行符(变成一行)

```
vim ingress-secret.yml

apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5akNDQWQ2Z0F3SUJBZ0lKQU5TR2dNNnYvSVd5TUEwR0NTcUdTSWIzRFFFQkJRVUFNQkl4RURBT0JnTlYKQkFNTUIydDFZbVV0WTJFd0hoY05NVGN3TXpBME1USTBPRFF5V2hjTk1UZ3dNekEwTVRJME9EUXlXakFYTVJVdwpFd1lEVlFRRERBeHJkV0psTFdsdVozSmxjM013Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUM2dkNZRFhGSFpQOHI5Zk5jZXlkV015VVlELzAwQ2xnS0M2WjNpYWZ0QlRDK005TmcrQzloUjhJUE4KWW00cjZOMkw1MmNkcmZvQnBHZXovQVRIT0NJYUhJdlp1K1ZaTzNMZjcxZEVLR09nV21LMTliSVAzaGpSeDZhWQpIeGhEVWNab3ZzYWY1UWJHRnUydEF4L2doMTFMdXpTZWJkT0Y1dUMrWHBhTGVzWWdQUjhFS0cxS0VoRXBLMDFGCmc4MjhUU1g2TXVnVVZmWHZ1OUJRUXExVWw0Q2VMOXhQdVB5T3lMSktzbzNGOEFNUHFlaS9USWpsQVFSdmRLeFYKVUMzMnBtTHRlUFVBb2thNDRPdElmR3BIOTZybmFsMW0rMXp6YkdTemRFSEFaL2k1ZEZDNXJOaUthRmJnL2NBRwppalhlQ01xeGpzT3JLMEM4MDg4a0tjenJZK0JmQWdNQkFBR2pTakJJTUM0R0ExVWRFUVFuTUNXQ0VtUmhjMmhpCmIyRnlaQzV0Y21sMFpDNXRaWUlQYTJsaVlXNWhMbTF5YVhSa0xtMWxNQWtHQTFVZEV3UUNNQUF3Q3dZRFZSMFAKQkFRREFnWGdNQTBHQ1NxR1NJYjNEUUVCQlFVQUE0SUJBUUNFN1ByRzh6MytyaGJESC8yNGJOeW5OUUNyYVM4NwphODJUUDNxMmsxUUJ1T0doS1pwR1N3SVRhWjNUY0pKMkQ2ZlRxbWJDUzlVeDF2ckYxMWhGTWg4MU9GMkF2MU4vCm5hSU12YlY5cVhYNG16eGNROHNjakVHZ285bnlDSVpuTFM5K2NXejhrOWQ1UHVaejE1TXg4T3g3OWJWVFpkZ0sKaEhCMGJ5UGgvdG9hMkNidnBmWUR4djRBdHlrSVRhSlFzekhnWHZnNXdwSjlySzlxZHd1RHA5T3JTNk03dmNOaQpseWxDTk52T3dNQ0h3emlyc01nQ1FRcVRVamtuNllLWmVsZVY0Mk1yazREVTlVWFFjZ2dEb1FKZEM0aWNwN0sxCkRPTDJURjFVUGN0ODFpNWt4NGYwcUw1aE1sNGhtK1BZRyt2MGIrMjZjOVlud3ROd24xdmMyZVZHCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdXJ3bUExeFIyVC9LL1h6WEhzblZqTWxHQS85TkFwWUNndW1kNG1uN1FVd3ZqUFRZClBndllVZkNEeldKdUsramRpK2RuSGEzNkFhUm5zL3dFeHpnaUdoeUwyYnZsV1R0eTMrOVhSQ2hqb0ZwaXRmV3kKRDk0WTBjZW1tQjhZUTFIR2FMN0duK1VHeGhidHJRTWY0SWRkUzdzMG5tM1RoZWJndmw2V2kzckdJRDBmQkNodApTaElSS1N0TlJZUE52RTBsK2pMb0ZGWDE3N3ZRVUVLdFZKZUFuaS9jVDdqOGpzaXlTcktOeGZBREQ2bm92MHlJCjVRRUViM1NzVlZBdDlxWmk3WGoxQUtKR3VPRHJTSHhxUi9lcTUycGRadnRjODJ4a3MzUkJ3R2Y0dVhSUXVhelkKaW1oVzRQM0FCb28xM2dqS3NZN0RxeXRBdk5QUEpDbk02MlBnWHdJREFRQUJBb0lCQUJtRmIzaVVISWVocFYraAp1VkQyNnQzVUFHSzVlTS82cXBzenpLVk9NTTNLMk5EZUFkUHhFSDZhYlprYmM4MUNoVTBDc21BbkQvMDdlQVRzClU4YmFrQ2FiY2kydTlYaU5uSFNvcEhlblFYNS8rKys4aGJxUGN6cndtMzg4K0xieXJUaFJvcG5sMWxncWVBOW0KVnV2NzlDOU9oYkdGZHh4YzRxaUNDdmRETDJMbVc2bWhpcFRKQnF3bUZsNUhqeVphdGcyMVJ4WUtKZ003S1p6TAplYWU0bTJDR3R0bmNyUktodklaQWxKVmpyRWoxbmVNa3RHODFTT3QyN0FjeDRlSnozbmcwbjlYSmdMMHcwU05ZCmlwd3I5Uk5PaDkxSGFsQ3JlWVB3bDRwajIva0JIdnozMk9Qb2FOSDRQa2JaeTEzcks1bnFrMHBXdUthOEcyY00KLzY4cnQrRUNnWUVBN1NEeHRzRFFBK2JESGdUbi9iOGJZQ3VhQ2N4TDlObHIxd2tuTG56VVRzRnNkTDByUm1uZAp5bWQ4aU95ME04aUVBL0xKb3dPUGRRY240WFdWdS9XbWV5MzFVR2NIeHYvWlVSUlJuNzgvNmdjZUJSNzZJL2FzClIrNVQ1TEMyRmducVd2MzMvdG0rS0gwc0J4dEM3U2tSK3Y2UndVQk1jYnM3c0dUQlR4NVV2TkVDZ1lFQXlaaUcKbDBKY0dzWHhqd1JPQ0FLZytEMlJWQ3RBVmRHbjVMTmVwZUQ4bFNZZ3krZGxQaCt4VnRiY2JCV0E3WWJ4a1BwSAorZHg2Z0p3UWp1aGN3U25uOU9TcXRrZW04ZmhEZUZ2MkNDbXl4ZlMrc1VtMkxqVzM1NE1EK0FjcWtwc0xMTC9GCkIvK1JmcmhqZW5lRi9BaERLalowczJTNW9BR0xRVFk4aXBtM1ZpOENnWUJrZGVHUnNFd3dhdkpjNUcwNHBsODkKdGhzemJYYjhpNlJSWE5KWnNvN3JzcXgxSkxPUnlFWXJldjVhc0JXRUhyNDNRZ1BFNlR3OHMwUmxFMERWZWJRSApXYWdsWVJEOWNPVXJvWFVYUFpvaFZ0U1VETlNpcWQzQk42b1pKL2hzaTlUYXFlQUgrMDNCcjQ0WWtLY2cvSlplCmhMMVJaeUU3eWJ2MjlpaWprVkVMRVFLQmdRQ2ZQRUVqZlNFdmJLYnZKcUZVSm05clpZWkRpNTVYcXpFSXJyM1cKSEs2bVNPV2k2ZlhJYWxRem1hZW1JQjRrZ0hDUzZYNnMyQUJUVWZLcVR0UGxKK3EyUDJDd2RreGgySTNDcGpEaQpKYjIyS3luczg2SlpRY2t2cndjVmhPT1Z4YTIvL1FIdTNXblpSR0FmUGdXeEcvMmhmRDRWN1R2S0xTNEhwb1dQCm5QZDV0UUtCZ0QvNHZENmsyOGxaNDNmUWpPalhkV0ZTNzdyVFZwcXBXMlFoTDdHY0FuSXk5SDEvUWRaOXYxdVEKNFBSanJseEowdzhUYndCeEp3QUtnSzZmRDBXWmZzTlRLSG01V29kZUNPWi85WW13cmpPSkxEaUU3eFFNWFBzNQorMnpVeUFWVjlCaDI4cThSdnMweHplclQ1clRNQ1NGK0Q5NHVJUmkvL3ZUMGt4d05XdFZxCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
kind: Secret
metadata:
  name: ingress-secret
  namespace: kube-system
type: Opaque
```

> kubectl create -f ingress-secret.yml

### 快速创建 Secret
> kubectl create secret tls ingress-secret --key cert/ingress-key.pem --cert cert/ingress.pem


### 重新部署

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-kibana-ingress
  namespace: kube-system
spec:
  tls:
  - hosts:
    - dashboard.mritd.me
    - kibana.mritd.me
    secretName: ingress-secret
  rules:
  - host: dashboard.mritd.me
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 80
  - host: kibana.mritd.me
    http:
      paths:
      - backend:
          serviceName: kibana-logging
          servicePort: 5601
```

**注意:**  
一个 Ingress 只能使用一个 secret(secretName 段只能有一个)，也就是说只能用一个证书，更直白的说就是如果你在一个 Ingress 中配置了多个域名，那么使用 TLS 的话必须保证证书支持该 Ingress 下所有域名；并且这个 secretName 一定要放在上面域名列表最后位置，否则会报错 `did not find expected key `无法创建；同时上面的 hosts 段下域名必须跟下面的 rules 中完全匹配

**更需要注意一点：**  
Kubernetes Ingress 默认情况下，当你不配置证书时，会默认给你一个 TLS 证书的，也就是说你 Ingress 中配置错了，比如写了2个 secretName、或者 hosts 段中缺了某个域名，那么对于写了多个 secretName 的情况，所有域名全会走默认证书；对于 hosts 缺了某个域名的情况，缺失的域名将会走默认证书，部署时一定要验证一下证书，不能 “有了就行”；更新 Ingress 证书可能需要等一段时间才会生效

## 参考文档
[Kubernetes Nginx Ingress 教程](https://mritd.me/2017/03/04/how-to-use-nginx-ingress/?utm_source=tuicool&utm_medium=referral)