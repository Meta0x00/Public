《记：Docker生产部署Nginx》
0x00:安装Docker && 配置镜像加速
```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```
```
vi  /etc/docker/daemon.json

{
"registry-mirrors":["https://registry.docker-cn.com"]
}
```
```
systemctl enable docker
systemctl restart docker
```
0x01:home目录下创建宿主机挂在卷
```
mkdir /home/nginx/
mkdir /home/nginx/logs/
mkdir /home/nginx/certs/
mkdir /home/nginx/conf.d/
mkdir /home/nginx/html/
touch /home/nginx/nginx.conf
touch /home/nginx/conf.d/default.conf
```
0x02:启动nginx容器
```
docker run -itd \
--restart=always \
--name nginx \
--privileged=true \
-p 80:80 \
-p 443:443 \
-e "TZ=Asia/Shanghai" \
-v /home/nginx/conf.d:/etc/nginx/conf.d \
-v /home/nginx/certs:/etc/nginx/certs \
-v /home/nginx/logs:/var/log/nginx \
-v /home/nginx/html:/home/www nginx \
/bin/bash
```
0x03:进入容器 启动nginx服务
```
docker exec -it nginx /bin/bash
cd /etc/init.d
./nginx start
```
0x04:安装依赖包
```
apt update \
&& apt upgrade \ 
&& apt install -y iptables-dev libipset-dev libnfnetlink-dev libnl-3-dev libnl-genl-3-dev libnl-route-3-dev libssl-dev lsof procps gcc make daemon net-tools vim
```
0x05:安装keepalived
```
apt install keepalived
```
0x06:编码chk_nginx.sh(置于/etc/keepalived/)
```
A=`ps -C nginx --no-header |wc -l`
if [ $A -eq 0 ];then
       service keepalived stop
fi
```
0x07:编码keepalived.conf(置于/etc/keepalived/)
```
! Configuration File for keepalived
global_defs {
        router_id 172.16.1.221
}
vrrp_script chk_nginx {
    script "/etc/keepalived/chk_nginx.sh"
    interval 2
    weight 2
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    mcast_src_ip 172.16.1.221
    virtual_router_id 172
    priority 100
    nopreempt
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        # 117.50.43.68
        172.16.0.1
    }
    track_script {
        chk_nginx
    }
}
```
0x08:启动Keepalived
```
service keepalived start
```
