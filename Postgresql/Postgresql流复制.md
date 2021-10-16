《基于Docker实现Postgresql流复制》
### 环境
```
    Ubuntu20.04
    Docker20.10.6
    Postgresql10
```
### 0x01 创建挂载卷
```
docker volume create pg_data
```
### 0x02 创建数据库容器
```
docker run -itd --name='PG' --restart always \
-e POSTGRES_PASSWORD='123456' \
-e POSTGRES_USER='postgres' \
-e ALLOW_IP_RANGE=0.0.0.0/0 \
--privileged=true \
-p 5432:5432 \
-v pg_data:/var/lib/postgresql/data/ \
-d postgres:xray
```
### 0x03 更新软件包
```
apt update && apt upgrade
```
### 0x04 创建备份文件(备库)
```
pg_basebackup -h 172.16.1.57 -U replica -F p -P -R -D /var/lib/postgresql/data/ -l rep_backup
```
### 0x05 主节点配置
```
1. 使用postgres用户登陆数据库，创建复制用户
	需要一个账号进行主从同步
	postgres=#create role replica login replication encrypted password 'replica';
2. 认证文件pg_hba.conf中加入standby节点权限
	host replication replica 172.16.1.32/32 trust
3. 主库配置文件postgresql.conf
	listen_addresses = '*'
	max_connections = 1000
	wal_level = replica
	max_wal_senders = 1
	wal_keep_segments = 64
	archive_mode = on
	archive_command = 'cp %p /var/lib/postgresql/data/pg_archive/%f'
	wal_log_hints = on
	full_page_writes = on
	hot_standby = on
4.创建pg_archive目录
	cd /var/lib/postgresql/data
	mkdir pg_archive
	chown -R postgres:postgres pg_archive
5.配置.pgpass
	su - postgres
	vim ~/.pgpass
	172.16.1.57:5432:replication:replica:replica
	172.16.1.32:5432:replication:replica:replica
6.配置recovery文件（master:recovery.done, standby:recovery.conf）
	vim recovery.done
		standby_mode = on
		primary_conninfo = 'host=172.16.1.32 port=5432 user=replica password=replica'
		recovery_target_timeline = 'latest'
7. 重启
```
### 0x06 备节点配置
```
1. 删除data下所有数据
	cd /var/lib/postgresql/data
	rm -rf *
2. 连接master节点，备份
	pg_basebackup -h 172.16.1.57 -U replica -F p -P -R -D /var/lib/postgresql/data/ -l rep_backup
	(密码为：replica)
3. 变更权限
	chown -R postgres:postgres /var/lib/postgresql/data
4. 修改recovery文件
	mv recovery.done recovery.conf
	vim recovery.conf
	standby_mode = on
	primary_conninfo = 'host=172.16.1.57 port=5432 user=replica password=replica'
	recovery_target_timeline = 'latest'
5. 创建pg_archive目录
	cd /var/lib/postgresql/data
	mkdir pg_archive
	chown -R postgres:postgres  /var/lib/postgresql/data/pg_archive
6. 配置.pgpass
	su - postgres
	vim ~/.pgpass
	172.16.1.57:5432:replication:replica:replica
	172.16.1.32:5432:replication:replica:replica
7. 重启
```
