>### https://github.com/lizhenliang/simple-microservice
>### 本地测试
1.导入db目录下数据库文件到自己的MySQL服务器
2.修改配置环境（xxx-service/src/main/resources/application.yml，active值决定启用环境配置文件）
3.修改连接数据库配置（xxx-service/src/main/resources/application-fat.yml）
4.修改前端页面连接网关地址（portal-service/src/main/resources/static/js/productList.js和orderList.js）
    原来: http://gateway.ctnrs.com/order/queryAllOrder
    改为: http://localhost:9999/order/queryAllOrder
5.服务启动顺序：eureka -> mysql -> product,stock,order -> gateway -> portal

>### 部署到K8S
1.安装基础环境
    jdk，maven,mysql
2.安装harbor镜像仓库
    https://blog.csdn.net/qq_39680564/article/details/97375772
3.安装运行ingress容器
    1).拉取镜像
    # 拉镜像
    docker pull registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64:1.5
    docker pull registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller:0.20.0
    # 打tag
    docker tag registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64:1.5 k8s.gcr.io/defaultbackend-amd64:1.5
    docker tag registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller:0.20.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.20.0
    # 删原标签
    docker rmi registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64:1.5
    docker rmi registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller:0.20.0

    2).安装ingress nginx服务
    kubectl apply -f mandatory.yaml
    kubectl apply -f service-nodeport.yaml

    3).安装完毕，先查看namespace，增加了一个ingress-nginx的专用命名空间：
    	kubectl get ns

    	ingress-nginx

    4).查看ingress-nginx命名空间中各pod服务运行状态：
    	kubectl get pods -n ingress-nginx

    NAME                                        READY   STATUS    RESTARTS   AGE
    default-http-backend-c858dffd-7w5qg         1/1     Running   0          2d
    nginx-ingress-controller-7cbb74c44b-fpn75   1/1     Running   0          2d

    5).再查看ingress-nginx命名空间中各service服务运行状态：
    	kubectl get services -n ingress-nginx

    NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
    default-http-backend   ClusterIP   10.98.98.189   <none>        80/TCP                       2d
    ingress-nginx          NodePort    10.105.56.57   <none>        80:32616/TCP,443:31398/TCP   45h

4.根据ingress的暴露端口改前端页面连接网关地址。http://gateway.ctnrs.com:32759/order/queryAllOrder
5.修改hosts文件(本机/服务器)
    192.168.217.130 eureka.ctnrs.com gateway.ctnrs.com portal.ctnrs.com
6.执行k8s目录的docker_build.sh脚本。实现镜像打包并上传到harbor仓库
7.修改yaml文件的镜像地址
  yaml服务启动顺序：eureka -> mysql -> product,stock,order -> gateway -> portal
8.地址: eureka: eureka.ctnrs.com:32759
        前端:   portal.ctnrs.com:32759