---
layout:     post
title:      云原生｜实战：安装K8s的dashboard，图形化还是挺香的
date:       2023-12-10
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - CLoud
    - 云原生
    - K8s
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

![][0]

![@七禾页话][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

当一个系统建立后，运维便要立即跟上，所以监控系统是必不可少的。

现在Kubernetes中的监控系统层出不穷，比如现在非常流行的`Prometheus`、`Grafana`、`Elastic Stack (ELK)`、` Kubernetes Metrics Server`等等，它们不仅仅用来监控系统，还用于追踪集群的健康状况、性能和资源利用率。

今天先不聊上边这些监控系统，想要聊的是kubernetes项目中原生的监控系统——kubernetes Dashboard，它提供了一个基于 Web 的用户界面，允许用户以图形化的方式管理和监控 Kubernetes 集群中的容器化应用。虽然大部分 Kubernetes 任务可以通过命令行工具`kubectl`完成，但 Dashboard 提供了几个关键的优势：

### 1. 可视化管理

- **直观操作**：Dashboard 提供了一个用户友好的界面，使得 Kubernetes 资源的创建、读取、更新和删除（CRUD）操作更加直观和易于理解。
- **实时状态查看**：它显示了关于集群和应用的实时状态信息，包括 Pods、Deployments、Services 等。

### 2. 简化复杂操作

- **图形化界面**：对于不熟悉命令行或 Kubernetes 资源文件（YAML）的用户来说，图形化界面可以简化许多复杂的操作。
- **快速入门**：对于新手来说，Dashboard 提供了学习 Kubernetes 概念和资源管理的快速途径。

### 3. 资源监控和故障排查

- **集中监控**：Dashboard 可以用来监控集群资源的使用情况，包括 CPU、内存使用量等。
- **日志查看**：用户可以直接在 Dashboard 中查看 Pod 日志，这对于快速定位问题非常有用。

### 4. 安全性和访问控制

- **基于角色的访问控制**：Dashboard 支持 Kubernetes 的 RBAC（基于角色的访问控制），允许对不同用户的访问权限进行精细控制。
- **多用户环境**：在多用户环境中，不同的用户可以根据他们的权限看到不同的视图和资源。

### 5. 插件和扩展

- **扩展性**：Dashboard 的设计允许集成额外的功能和插件，为用户提供更多的灵活性。

说了这么多，那么如何安装这个dashboard呢？可以说联网的kubernetes中任何组件的安装都非常容易。

#### 安装dashbaord

这个dashboard依然继承了一条命令即可安装的优良传统：
```bash
###左右滑动
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

例子
ubuntu@master:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
ubuntu@master:~$ 
```

可以看到运行完apply之后，会创建一个叫做`kubernetes-dashboard`的namespace，然后创建相关的`service`、`secret`、`configmap`以及重要的`deployment`等相关内容。

启动很快，大概不到一分钟的时间，POD就已经是Ready的状态了，默认创建的service都是ClusterIP类型：
```bash
###左右滑动
ubuntu@master:~$ kubectl get pods,svc -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-5cb4f4bb9c-x827t   1/1     Running   0          41s
pod/kubernetes-dashboard-6967859bff-hqcqr        1/1     Running   0          41s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes-dashboard        ClusterIP   10.43.236.108   <none>        443/TCP    41s
service/dashboard-metrics-scraper   ClusterIP   10.43.174.125   <none>        8000/TCP   41s
ubuntu@master:~$ 
```

#### 如何登录GUI？

为了直接登录其GUI，需要对`kubernetes-dashboard`这个`service`进行改造，将`type`改成`NodePort`，改造完成后可以看到其类型是NodePort，并且分配了一个外部Port`443:30306/TCP`中的`30306`:
```bash
###左右滑动
kubectl patch svc kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]' -n kubernetes-dashboard

例子
ubuntu@master:~$ kubectl patch svc kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]' -n kubernetes-dashboard
service/kubernetes-dashboard patched
ubuntu@master:~$ 
ubuntu@master:~$ 
ubuntu@master:~$ kubectl get svc -n kubernetes-dashboard 
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.43.174.125   <none>        8000/TCP        92s
kubernetes-dashboard        NodePort    10.43.236.108   <none>        443:30306/TCP   92s
ubuntu@master:~$ 
```

此时登录GUI还差最后一个信息，那就是credentials：登录的验证信息。Dashboard的登录有两种方式，一个是token，一个是kubeconfig，相对来说token比较简单，所以实验中直接用token了。

通过创建ServiceAccount，并创建一个临时的token来作为登录GUI的Credential。
```bash
###左右滑动
cat >k3s-dashboard.yaml<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

kubectl create -f k3s-dashboard.yaml

kubectl -n kubernetes-dashboard create token admin-user

例子：
ubuntu@master:~$ kubectl -n kubernetes-dashboard create token admin-user
eyJhbGciOiJSUzI1NiIsImtpZCI6InNLVzY0OHVvZXgyU0pfTXNpRG8zejR2ZUNXbVNrQlp0d0JZbm1lMFF5T1EifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNzAyMTM2MzE4LCJpYXQiOjE3MDIxMzI3MTgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiMTBmMmQzNTEtMzdmMC00ZTRiLWEwYWMtMjhmNzJlZWVkN2VmIn19LCJuYmYiOjE3MDIxMzI3MTgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbi11c2VyIn0.u_Iz8nNF7_HmaTZR2BbjXOo9es6sfdJnmCT6znXfy2tHek-wJyfUnHvUBbGF1efZnEBAk6Z518DWDU88E2Us9Okq7oX4AcVe_Rgk_b78Z2KxgzOMhXjFf4mhY0H9DymNA_Qow_3E4ylU4AM0oumHBHTC6B7kUhnQxSdCVAR5NpKoaLdj3eEY0Bd7ckDr9zJ6sRZYsAJfCZffBug0XSXcxIp1m3QATuW4D18FZo4mMIRfwuive2B48y3QyTDyPkCG6VKw-SYYD_Yj-Oyl2bdvAPCKvjKEfv87Ke_f-hIpgekpevbSokk7xw2hhqrFSQTKAkxslDMCg96JHLaPKFSThg
ubuntu@master:~$ 
```

万事俱备，开始登录GUI，IP是如何一个node(包括master、worker)的IP，Port就是SVC里的外部Port，Credential就是上边打印出来的token：
![@七禾页话][3]

将namespace选成All namespaces:
![@七禾页话][4]

Deployment的统计信息：
![@七禾页话][5]

集群中的Nodes信息，可以看到master赫然在列，所以一直强调master是一种带有controller角色的worker node：
![@七禾页话][6]

到此，kubernetes的dashboard就完成了。

#### 登录的改进

为什么会有这一张呢？原因是登录dashboard的时候要临时获取一个token，每次登录都要填写一个长串，费时费力费事，所以我就在想如何可以通过用户名密码的方式随时登录dashboard的GUI呢？

所以这个是我走的野路子，不是kubernetes的官方推荐(它也没有啥推荐)，只能实验用一用，拓展一下思路。

我的方案是通过Nginx做反向代理，设置Nginx的登录用户名和密码，然后反向代理登录到kubernetes的dashboard上，具体做法如下：

先将dashboard的token永久化，之前的步骤中token是有期限的，过期就得重新获取一个token，显然是不行的，所以需要将其永久化。
```bash 
###左右滑动

##yaml文件
cat >k3s-dashboard-secret.yaml<<EOF
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
EOF

### apply yaml
kubectl apply -f k3s-dashboard-secret.yaml 

###获取永久化的token
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d;echo

例子：
ubuntu@master:~$ kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d;echo
eyJhbGciOiJSUzI1NiIsImtpZCI6InNLVzY0OHVvZXgyU0pfTXNpRG8zejR2ZUNXbVNrQlp0d0JZbm1lMFF5T1EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJhMWVmNTBiNi0zNjYyLTRhN2UtYTBlNi0xZWNkM2E0NWI1NDIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.BI08SEMjg6W-234DIg_sTGVU4Z9zM9FcO0qhSfdumMDtT7sw6ddhYc55FUaXNdp10vCRktFSLbGAJ4UR3z1TPXKikZ4T1pS1cVlviTxX4nJHw9FdiSvsZLmhSB9hDMiPtrL6Mu_C8i6z3VqTgjEGgwv01ZUpGhYPJPmPbzurUNheJxj6Zf5LZcRyfcbwGPhZSmgx7aTK2Z-HNjXGF5tofBoCcN-XYUE9J-mIXZB77m3vz3kA8ar-FTRDo1k8RAc6wdgIhSzGz8fjVXA4SLezHzlo9o6Hj8RPfmAey2Bfzuxz76pXQOP1rah6oB7LjZeGXyqMiKgkKaI2xxMT0uc7bQ
ubuntu@master:~$  
```

接下来是ngnix的操作部分。

首先需要通过`htpasswd`给用户`admin`创建密码文件
```bash 
###左右滑动
#因为我的template vm没有安装apache，所以要先安装一下
sudo apt install -y apache2-utils

# 用户名admin，密码admin，用来登录Nginx，密码文件的名字叫做nginx_htpasswd
htpasswd -cBb nginx_htpasswd admin admin
```

创建一个包含上边创建的密码文件的`secret`:
```bash
###左右滑动
kubectl create ns nginx

kubectl create secret -n nginx generic basic-auth --from-file=nginx_htpasswd
```

接下来的内容非常重要，就是定义nginx反向代理的`configmap`，其中`nginx.conf`立即的几个参数需要修改成自己实验中的值：
1. proxy_pass：IP和Port就是之前登录dashboard所使用的IP和Port即可，当然也可以使用`kubernetes-dashboard`这个`Service`的`CLUSTER-IP`和内部端口默认的`443`。
2. proxy_set_header的构成是`proxy_set_header Authorization "Bearer <永久性token>`。
3. auth_basic_user_file是固定的`/etc/nginx/.htpasswd`即可。
```bash 
###左右滑动
cat >nginx-configmap.yaml<<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: nginx
data:
  nginx.conf: |
    events {}
    http {
        server {
            listen 80;
            location / {
                proxy_pass https://192.168.100.106:30306;
                proxy_set_header Host \$http_host;
                proxy_set_header X-Real-IP \$remote_addr;
                proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto \$scheme;
                proxy_set_header Authorization "Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6InNLVzY0OHVvZXgyU0pfTXNpRG8zejR2ZUNXbVNrQlp0d0JZbm1lMFF5T1EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJhMWVmNTBiNi0zNjYyLTRhN2UtYTBlNi0xZWNkM2E0NWI1NDIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.BI08SEMjg6W-234DIg_sTGVU4Z9zM9FcO0qhSfdumMDtT7sw6ddhYc55FUaXNdp10vCRktFSLbGAJ4UR3z1TPXKikZ4T1pS1cVlviTxX4nJHw9FdiSvsZLmhSB9hDMiPtrL6Mu_C8i6z3VqTgjEGgwv01ZUpGhYPJPmPbzurUNheJxj6Zf5LZcRyfcbwGPhZSmgx7aTK2Z-HNjXGF5tofBoCcN-XYUE9J-mIXZB77m3vz3kA8ar-FTRDo1k8RAc6wdgIhSzGz8fjVXA4SLezHzlo9o6Hj8RPfmAey2Bfzuxz76pXQOP1rah6oB7LjZeGXyqMiKgkKaI2xxMT0uc7bQ";

                auth_basic "Restricted Access";
                auth_basic_user_file /etc/nginx/.htpasswd;
            }
        }
    }
EOF

kubectl apply -f nginx-configmap.yaml 

kubectl get configmaps -n nginx 
```

接下来是通过deployment这个controller创建Ngnix的Pod，其中`volumeMounts`下的两个内容非常重要。一个是配置文件，一个是登录ngnix的用户名和密码，具体deployment的yaml如下，准备完直接apply后等待Pod ready即可：
```bash
###左右滑动
cat >nginx-deployment.yaml<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy
  namespace: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-proxy
  template:
    metadata:
      labels:
        app: nginx-proxy
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: auth-volume
          mountPath: /etc/nginx/.htpasswd
          subPath: nginx_htpasswd
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
      - name: auth-volume
        secret:
          secretName: basic-auth
EOF

kubectl apply -f nginx-deployment.yaml 

kubectl get pod -n nginx 
```

与此同时，需要给nginx创建一个`Service`，并将`Service`暴漏给外部可以登录，目前的做法是将`service`的`type`改成`NodePort`，同时也将暴漏的外部Port固定成了`30000`，前提是`30000`没有被其他的`service`占用：
```bash
###左右滑动
cat >nginx-service.yaml<<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-proxy-service
  namespace: nginx
spec:
  selector:
    app: nginx-proxy
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF

kubectl apply -f nginx-service.yaml 

kubectl get svc -n nginx

kubectl patch svc -n nginx nginx-proxy-service --type='json' -p='[{"op": "replace", "path": "/spec/type", "value": "NodePort"}, {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30000}]'
```

通过上述步骤后，就可以使用某一个node的ip和Nginx Service的外部Port `30000`登录，会提示输入用户名和密码，需要注意的是本实验需要使用http的协议登录nginx，即`http://<node ip>:30000`：
![@七禾页话][6]

使用正确的用户名密码登录后会自动跳转到kubernetes dashboard：
![@七禾页话][7]

以上就是安装dashboard的流程，并进行一个nginx方向代理的使用，如果又更好的方案或有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAYJ9EJ3fj2uXNCliamgQMe6ySicVtlFUoXTfZ3eFZDiaf67JwAR0yZ6ZibtFEBKz6y4MCz8pcNaefX2w/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLiahdtHyzGPk1enKcT3hz5bBAXgIU5wAialCqXWhQDLaGIfyOef3R6wZrA/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLia3AspkrT8ic1O7Db4eVFgo2nS1ZRAjL2JhxG3TKSkvoeD9CwKr8teibCQ/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLiagw77FHsSwNFqm02kTl4Rbbz1DAOIKXTpBXBwHbulxecFCHmz2MLXJg/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLiaa2cO7Af9jo6d9hicJmsYhibwb0E9Q7ibaOk4YeY7h2iaY3icFpdia2E7NNIw/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLiaMTrwyn4PIGmUStLwia9lqallZic1Doj0RCym96U9xun2MUU9SmQx4BdA/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLiaKKx15fHEyibDGvD840icCp1cdebr6ibIyX8jpMZ9Kl2M1CYxvfVg4c1OQ/640?wx_fmt=png&amp;from=appmsg


