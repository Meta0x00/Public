##### <font color=red>(若图片无法加载，请配置本地hosts文件，重新声明DNS，......或者直接科学上网！)</font>
# 记：基于Docker搭建postgresql数据库
### 一、拉取postgres镜像
![alt postgresl_01](./img/postgresql_01.png) 
```
docker pull postgres
```
### 二、构建镜像容器
![alt postgresl_05](./img/postgresql_05.png) 
```
docker volume create dv_pgdata
```
![alt postgresl_02](./img/postgresql_02.png) 
```
docker run -itd --name='PG' --restart always \
> -e POSTGRES_PASSWORD=abc \
> -e ALLOW_IP_RANGE=0.0.0.0/0 \
> -p 5432:5432 \
> -p 9187:9187 \
> -v /root/prometheus/:/root/prometheus/ \
> -v dev_pgdata:/var/lib/postgresql/data/ \
> -d postgres:12.1
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
### 五、配置prometheus监控PG数据库
![alt postgresl_06](./img/postgresql_06.png) 
```
tar xf postgres_exporter-0.9.0.linux-amd64.tar.gz -C /opt/
```
![alt postgresl_07](./img/postgresql_07.png) 
```
cd /opt/postgres_exporter-0.9.0.linux-amd64/
```
![alt postgresl_08](./img/postgresql_08.png) 
```
vim start.sh
```
![alt postgresl_09](./img/postgresql_09.png) 
```
#!/bin/bash

cd /opt/postgres_exporter-0.9.0.linux-amd64
export PATH=/usr/local/pgsql/bin:$PATH

# 下面的password替换为你自己的postgres用户的密码
export  DATA_SOURCE_NAME="user=postgres host=localhost password=abc port=5432 dbname=postgres sslmode=disable"

nohup ./postgres_exporter --web.listen-address=":9187" >/dev/null 2>&1 &
```
![alt postgresl_10](./img/postgresql_10.png) 
```
vim /usr/local/prometheus/prometheus.yml
```
![alt postgresl_11](./img/postgresql_11.png)
```
- job_name: 'pg'
    static_configs:
    - targets: ['152.32.170.211:5432']
- job_name: 'pg1'
    static_configs:
    - targets: ['152.32.170.211:9187']
```
![alt postgresl_12](./img/postgresql_12.png)
```
pkill prometheus
```
![alt postgresl_13](./img/postgresql_13.png)
```
/usr/local/prometheus/prometheus --config.file="/usr/local/prometheus/prometheus.yml" &
```
![alt postgresl_14](./img/postgresql_14.png)
