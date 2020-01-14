# my-ansible-elk
## ansible实战之部署elk集群
项目已放入github：https://github.com/hqq-get/my-ansible-elk

### 规划
主机名 | IP | 功能
---|---|---
ansible | 192.168.159.21 | ansible管理机
es-node1 | 192.168.159.30 | es kibana
es-node2 | 192.168.159.31 | es kibana
es-node3 | 192.168.159.32 | es kibana
logstash | 192.168.159.33 | logstash
db | 192.168.159.34 | redis filebeat nginx
filebeat |192.168.159.35| filebeat
## 项目目录结构
```
.
├── es.yml
├── filebeat.yml
├── group_vars
│   ├── all
│   │   └── main.yml
│   ├── redis
│   ├── render-node-alpha
│   └── render-tools-alpha
├── invertories
│   └── hosts
├── kibana.yml
├── logstash.yml
├── nginx.yml
├── redis.yml
└── roles
    ├── common
    │   ├── files
    │   │   └── elk.repo
    │   └── tasks
    │       └── main.yml
    ├── es
    │   ├── files
    │   │   └── elasticsearch.yml.j2
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   ├── elasticsearch.yml.j2
    │   │   └── jvm.options.j2
    │   └── vars
    │       └── main.yml
    ├── filebeat
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   └── filebeat.yml.j2
    │   └── vars
    ├── kibana
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   └── kibana.yml.j2
    │   └── vars
    ├── logstash
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   └── my_elk.conf.j2
    │   └── vars
    ├── nginx
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   └── kibana.conf.j2
    │   └── vars
    │       └── main.yml
    └── redis
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        ├── tasks
        │   └── main.yml
        ├── templates
        │   └── redis.conf.j2
        └── vars
            └── main.yml.bak

49 directories, 36 files
```
### 配置ssh免密登录
```bash
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.159.30

ssh-copy-id -i .ssh/id_rsa.pub root@192.168.159.31

ssh-copy-id -i .ssh/id_rsa.pub root@192.168.159.32

ssh-copy-id -i .ssh/id_rsa.pub root@192.168.159.33

sh-copy-id -i .ssh/id_rsa.pub root@192.168.159.34

sh-copy-id -i .ssh/id_rsa.pub root@192.168.159.35
```
### hosts
根据实际情况修改
```
root@ansible roles]# cat invertories/hosts 
[redis]
192.168.159.34

[es]
192.168.159.30
192.168.159.31
192.168.159.32

[logstash]
192.168.159.33

[renderpackageservice-alpha]
192.168.159.34

[render-node-service-alpha]
192.168.159.34

[render-tools-alpha:children]
renderpackageservice-alpha
render-node-service-alpha

[render-node-alpha]
192.168.159.35

[nginx]
192.168.159.34
```
### 如何执行

```
ansible-playbook -i ./invertories/hosts redis.yml
ansible-playbook -i ./invertories/hosts es.yml
ansible-playbook -i ./invertories/hosts logstash.yml
ansible-playbook -i ./invertories/hosts filebeat.yml
ansible-playbook -i ./invertories/hosts filebeat.yml
ansible-playbook -i ./invertories/hosts --extra-vars "host=render-node-alpha" filebeat.yml
```
### 特别说明group_vars的使用
group_vars的作用是分文件定义 Host 和 Group 变量

在 inventory 主文件中保存所有的变量并不是最佳的方式.还可以保存在独立的文件中,这些独立文件与 inventory 文件保持关联. 不同于 inventory 文件(INI 格式),这些独立文件的格式为 YAML。

假设 inventory 文件的路径为:


```
/etc/ansible/hosts
```

假设有一个主机名为 'foosball', 主机同时属于两个组,一个是 'raleigh', 另一个是 'webservers'. 那么以下配置文件(YAML 格式)中的变量可以为 'foosball' 主机所用.依次为 'raleigh' 的组变量,'webservers' 的组变量,'foosball' 的主机变量:
```bash
/etc/ansible/group_vars/raleigh
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

举例来说,假设你有一些主机,属于不同的数据中心,并依次进行划分.每一个数据中心使用一些不同的服务器.比如 ntp 服务器, database 服务器等等. 那么 'raleigh' 这个组的组变量定义在文件 '/etc/ansible/group_vars/raleigh'之中,可能类似以下这样，类似本例的group_vars下的reids和render-node-alpha等


```
---
ntp_server: acme.example.org
database_server: storage.example.org
```
还有更进一步的运用,你可以为一个主机,或一个组,创建一个目录,目录名就是主机名或组名.目录中的可以创建多个文件, 文件中的变量都会被读取为主机或组的变量.如下'raleigh'主机组对应于 /etc/ansible/group_vars/raleigh/ 目录,其下有两个文件 db_settings 和 cluster_settings, 其中分别设置不同的变量。 
```
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```
如本例中group_vars下的all目录，对应主机组all也就是所有主机的变量设定，即全局变量。当全局变量和局部主机组变量冲突的时候，ansible会以局部主机组变量为准。即本例中all和redis都有redispass的变量，以./group_vars/redis为准

