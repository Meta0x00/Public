##### <font color=red>(若图片无法加载，请配置本地hosts文件，重新声明DNS，......或者直接科学上网！)</font>
# 记：基于Docker搭建postgresql数据库
### 一、拉取postgres镜像
![alt postgresl_01](./img/postgresql_01.png) 
```
docker pull postgres
```
### 二、构建镜像容器
![alt postgresl_02](./img/postgresql_02.png) 
```
docker run -itd --name='PG' --restart always \
> -e POSTGRES_PASSWORD='abc123' \
> -e ALLOW_IP_RANGE=0.0.0.0/0 \
> --network host \
> -v /root/prometheus/:/root/prometheus/ \
> <CONTAINER_ID> \
> /bin/bash
```
### 三、进入容器
![alt postgresl_03](./img/postgresql_03.png) 
```
docker exec -it PG /bin/bash
```
### 四、进入postgresql数据库
![alt postgresl_04](./img/postgresql_04.png) 
```
psql -U postgres
```
