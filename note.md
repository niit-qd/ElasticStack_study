[TOC]


---

### 5-05-集群基础环境准备-720P 高清-AVC

1. 配置hostname
    ``` shell
    hostnamectl set-hostname NAME
    ```
    ``` shell
    # 分别配置
    # mv /etc/hostname /etc/hostname.bak
    hostnamectl set-hostname elk-101
    hostnamectl set-hostname elk-102
    hostnamectl set-hostname elk-103
    ```
2. 设置hosts
    ``` shell
    cat >> /etc/hosts/ <<`EOF`
    192.168.65.101 elk-101
    192.168.65.102 elk-102
    192.168.65.103 elk-103
    EOF
    ```
    ``` shell
    # cp /etc/hosts /etc/hosts.bak
    vim /etc/hosts

    192.168.65.101 elk-101
    192.168.65.102 elk-102
    192.168.65.103 elk-103
    ```
3. 关闭防火墙
    ``` shell
    systemctl disable firewalld
    systemctl stop firewalld
    ```
4. 禁用SELINUX
    ``` shell
    setenforce 0
    getenforce
    ```
5. 生成密钥
    ``` shell
    ssh-keygen -t rsa
    ```
    ``` shell
    # 拷贝密钥 免密
    ssh-copy-id elk-101
    ssh-copy-id elk-102
    ssh-copy-id elk-103
    ```
6. 数据同步工具
    ``` shell
    dnf install rsync -y
    ```
    ``` shell
    vim /usr/local/sbin/data_rsync.sh # 将下面的内容拷贝到该文件即可
    chmod +x /usr/local/sbin/data_rsync.sh
    ```
    ``` shell
    #! /bin/bash
    # Auther: Jason Yin
    if [ $# -ne 1 ];then
        echo "Usage: $o /path/to/file (绝对路径)"
        exit
    fi
    
    # 判断文件是否存在
    if [ ! -e $1 ];then
        echo "[ $1 ] dir or file not find!"
        exit
    fi
    
    # 获取父路径
    fullpath=`dirname $1`
    
    # 获取子路径
    basename=`basename $1`

    # 进入到父路径
    cd $fullpath

    # for ((host_id=102;host_id<=103;host_id++))
    host_ids=("101" "102" "103")
    for host_id in ${host_ids[@]}
    do
        host=elk-$host_id
        if [ $host == $(hostname) ]; then
            continue
        fi
        # 使得终端输出变为绿色
        tput setaf 2
        echo ===== rsyncing $host: $basename =====
        # 使得终端恢复原来的颜色
        tput setaf 7
        # 将数据同步到其他两个节点
        rsync -az $basename `whoami`@$host:$fullpath
        if [ $? -eq 0 ];then
            echo "命令执行成功!"
        fi
    done
   ``` 
7. 集群时间同步
    ``` shell
    # (1)安装常用的Linux工具，您可以自定义哈。
    yum -y install vim net-tools
    # (2)安装chrony服务
    yum -y install ntpdate chrony
    # (3)修改chrony服务配置文件
    vim /etc/chrony.conf
    # 注释官方的时间服务器，换成国内的时间服务器即可
    server ntp.aliyun.com iburst
    server ntpl.aliyun.com iburst
    server ntp2.aliyun.com iburst
    server ntp3.aliyun.com iburst
    server ntp4.aliyun.com iburst
    server ntp5.aliyun.com iburst
    ...
    systemctl restart chronyd
    # (4)配置chronyd的开机自启动
    systemctl enable --now chronyd
    # (5)查看服务
    systemctl status chronyd
    ```

---


### 6-06-ES单节点部署实战案例-720P 高清-AVC
---


1. 下载Elasticsearch 
    当前使用版本7
    [Elasticsearch 7.17.21](https://www.elastic.co/downloads/past-releases/elasticsearch-7-17-21)
2. 安装Elasticsearch
    当前使用rpm版本
    ``` shell
    dnf install -y elasticsearch-7.17.21-x86_64.rpm
    
    ### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch.service
    ### You can start elasticsearch service by executing
    sudo systemctl start elasticsearch.service

    Created elasticsearch keystore in /etc/elasticsearch/elasticsearch.keystore
    ```
3. 启用Elasticsearch
    ``` shell
    systemctl start elasticsearch.service
    systemctl status elasticsearch.service
    # 查看端口 9200, 9300
    ss -ntl
    LISTEN     0      128                                           [::ffff:127.0.0.1]:9200                                                                    [::]:*
    LISTEN     0      128                                                        [::1]:9200                                                                    [::]:*
    LISTEN     0      128                                           [::ffff:127.0.0.1]:9300                                                                    [::]:*
    LISTEN     0      128                                                        [::1]:9300                                                                    [::]:*
    ```

4. 配置Elasticsearch
    ``` shell
    ll /etc/elasticsearch/
    -rw-rw----. 1 root elasticsearch  3429 Apr 26 00:39 elasticsearch.yml
    -rw-rw----. 1 root elasticsearch  3329 Apr 26 00:39 jvm.options
    ```
    1. elasticsearch.yml
        ``` yml
        cluster.name: my-elk-cluster
        node.name: node-elk-101
        network.host: 0.0.0.0
        discovery.seed_hosts: ["elk-101", "elk-102", "elk-103"]
        ```
        ``` shell
        systemctl restart elasticsearch.service
        ```
        ``` shell
        ss -ntl
        [root@elk-101 Elasticsearch]# ss -ntl
        State      Recv-Q Send-Q                                             Local Address:Port                                                            Peer Address:Port
        LISTEN     0      128                                                         [::]:9200                                                                    [::]:*
        LISTEN     0      128                                                         [::]:9300                                                                    [::]:*
        ```
        日志文件，`/var/log/elasticsearch/${cluster-name}.log`
        ``` log
        vim /var/log/elasticsearch/my-elk-cluster.log

        [2024-05-05T12:11:55,646][INFO ][o.e.h.AbstractHttpServerTransport] [node-elk-101] publish_address {192.168.65.101:9200}, bound_addresses {[::]:9200}
        [2024-05-05T12:11:55,648][INFO ][o.e.n.Node               ] [node-elk-101] started
        ```
        ``` log
        [root@elk-101 Elasticsearch]# curl localhost:9200
        {
        "name" : "node-elk-101",
        "cluster_name" : "my-elk-cluster",
        "cluster_uuid" : "KEOZ3y2RRh-fMLqd7GwlBQ",
        "version" : {
            "number" : "7.17.21",
            "build_flavor" : "default",
            "build_type" : "rpm",
            "build_hash" : "d38e4b028f4a9784bb74de339ac1b877e2dbea6f",
            "build_date" : "2024-04-26T04:36:26.745220156Z",
            "build_snapshot" : false,
            "lucene_version" : "8.11.3",
            "minimum_wire_compatibility_version" : "6.8.0",
            "minimum_index_compatibility_version" : "6.0.0-beta1"
        },
        "tagline" : "You Know, for Search"
        }
        ```

---

### 7-07-ES集群部署及排错实战案例-720P 高清-AVC

---

1. 指定集群的初始主节点列表
    ``` yml
    #cluster.initial_master_nodes: ["node-1", "node-2"]
    cluster.initial_master_nodes: ["elk-101", "elk-102", "elk-103"]
    ```


---

### 8-08-kibana部署并对接ES集群-720P 高清-AVC

---

1. 下载 Kibana
    [Past Releases](https://www.elastic.co/cn/downloads/past-releases#kibana)
    Kibana 7.17.21
2. 配置 Kibana
    `/etc/kibana/kibana.yml`
    ``` yml
    #server.port: 5601

    #server.host: "localhost"
    server.host: "elk-101"
    server.host: "0.0.0.0"

    #server.name: "your-hostname"

    #elasticsearch.hosts: ["http://localhost:9200"]
    elasticsearch.hosts: ["http://elk-101:9200", "http://elk-102:9200", "http://elk-103:9200"]

    #i18n.locale: "en"
    i18n.locale: "zh-CN"
    ```

3. 启动 Kibana
    ```shell
    systemctl start kibana
    journalctl -u kibana
    
    ss -ntl
    LISTEN     0      128                 *:5601                            *:*
    ```

---

### 9-09-filebeat环境部署，工作原理-720P 高清-AVC

1. 下载 Filebeat
    [Past Releases](https://www.elastic.co/cn/downloads/past-releases#filebeat)
    Filebeat 7.17.21
2. 安装
    ``` shell
    yum -y localinstall filebeat-7.17.21-x86_64.rpm
    ```
    ``` shell
    filebeat --help
    # 查看配置
    systemctl cat filebeat
    ```
3. 配置 Filebeat
    [Filebeat Reference](https://www.elastic.co/guide/en/beats/filebeat/7.17/index.html)
    `/etc/filebeat/filebeat.yml`
    ``` yml
    
    ```
4. 参考文档
    1. 参考：
        [Filebeat Reference](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)
        - [Configure](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html)
          - [Inputs](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html)
            - [Log input](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html)
          - [Output](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-output.html)
            - [Console](https://www.elastic.co/guide/en/beats/filebeat/current/console-output.html)

    2. 实例
        - stdin ---> console
            ``` shell
            vim filebeat_01.yml
            ```
            ``` yml
            filebeat.inputs:
            - type: stdin
            
            output.console:
              pretty: true
            ```
            ```shell
            filebeat -e -c filebeat_std-to-console.yml
            ```
            参数：
            ``` txt
              -c, --c string                     Configuration file, relative to path.config (default "filebeat.yml")
              -e, --e                            Log to stderr and disable syslog/file output
                  --path.config string           Configuration path (default "")
            ```
            > `-c`参数指定了配置文件路径，它的目录是相对于`path.config`而定的。
            > 通过`filebeat export config`命令可知，`path.config`的默认目录是`/etc/filebeat`。
            > 如果希望使用当前目录下的配置文件，则应动态修改`path.config`的值。具体执行命令是：
            > `filebeat --path.config . -e -c xxx.yml`

            通过stdin控制台输入内容，可以生成信息（例如`message`字段），并在控制台中输出内容。
            ``` log
            2024-06-02T06:47:05.163-0400    ERROR   file/states.go:125      State for  should have been dropped, but couldn't as state is not finished.
            HelloWorld
            {
            "@timestamp": "2024-06-02T10:47:13.769Z",
            "@metadata": {
                "beat": "filebeat",
                "type": "_doc",
                "version": "7.17.21"
            },
            "host": {
                "name": "elk-101"
            },
            "agent": {
                "ephemeral_id": "e821c176-b084-401f-bbc1-a5ccc34b9864",
                "id": "576818cb-c407-4c7d-a2d8-6d8a102b3af6",
                "name": "elk-101",
                "type": "filebeat",
                "version": "7.17.21",
                "hostname": "elk-101"
            },
            "log": {
                "offset": 0,
                "file": {
                "path": ""
                }
            },
            "message": "HelloWorld",
            "input": {
                "type": "stdin"
            },
            "ecs": {
                "version": "1.12.0"
            }
            }
            2024-06-02T06:47:14.774-0400    ERROR   file/states.go:125      State for  should have been dropped, but couldn't as state is not finished.
            ```
        - 
    3. 

---

### 10-10-log输入类型及其工作原理解析-720P 高清-AVC

1. 示例
    [Log](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html)
    input: log-to-console
    ``` yml
    filebeat.inputs:
    - type: log
    paths:
        - tmp/test.log

    output.console:
    pretty: true
    ```
    ``` shell
    filebeat --path.config . -e -c filebeat-log-to-console.yml
    ```
    不断向测试日志文件中写入数据，filebeat会按行输出新修改的内容。
    当重新启动filebeat执行当前命令的时候，会发现filebeat并没有重新读取测试文件，其原理是filebeat将测试文件的读取偏移量offset写入到了记录文件中。
    该记录文件可以从当前运行输出的信息中找到。
    ``` log
    2024-06-08T12:23:32.774-0400    INFO    memlog/store.go:124     Finished loading transaction log file for '/var/lib/filebeat/registry/filebeat'. Active transaction id=82
    ```
    查看`/var/lib/filebeat/registry/filebeat`下的`log.json`，可以找到偏移量。关键字是`offset`。例如：
    ``` json
    {"k":"filebeat::logs::native::34333784-64768","v":{"offset":19,"timestamp":[280444960220998,1717863812],"FileStateOS":{"inode":34333784,"device":64768},"identifier_name":"native","type":"log","id":"native::34333784-64768","prev_id":"","source":"/root/demo/filebeat/tmp/test.log","ttl":-1}}
    ```
---

### 11-11-filebeat配置多个log输入实例-720P 高清-AVC

1. 示例
    log
    路径通配符 `*`

---

### 12-12-filebeat的input插件的通用字段-720P 高清-AVC

1. input 通用配置
    所有input类型都有的配置项
    Common options:
    - `enable`
      Use the enabled option to enable and disable inputs. By default, enabled is set to true.
      启用，默认`true`
    - `tags`
      A list of tags that Filebeat includes in the tags field of each published event.
      格式为数组列表
      可以理解为一个分类用的标志位。默认为空（生成的json数据不存在该字段）。 
    - `fields`
      Optional fields that you can specify to add additional information to the output.
      是一种额外信息。
      格式是json键值对。
    - `fields_under_root`
      If this option is set to true, the custom fields are stored as top-level fields in the output document instead of being grouped under a fields sub-dictionary.
      会将原先放在`fields`中的字段放到整个json的根字段中。
      当与默认的根字段同名时，会覆盖原始的值。

---

### 13-13-filebeat的output类型es类型案例-720P 高清-AVC

1. Elasticsearch
    [Elasticsearch](https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html)
    ``` yml
    filebeat.inputs:
    - type: log
    paths:
        - tmp/test.log

    output.elasticsearch:
    hosts: ["http://elk-101:9200", "http://elk-102:9200", "http://elk-103:9200"] 
    ```
    ``` shell
    filebeat --path.config . -e -c filebeat-log-to-elasticsearch.yml 
    ```
2. Kibana 查看索引路径
    `Stack Management` --> `索引管理`
    `http://elk-101:5601/app/management/data/index_management/indices`
    发现生成了一个索引，例如：filebeat-7.17.21-2024.06.08-000001。
    文档计数：存了多少条数据。

3. Kibnan index pattern 索引模式
    [Create an index pattern](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html#settings-create-pattern)
    Kibana requires an index pattern to access the Elasticsearch data that you want to explore. An index pattern selects the data to use and allows you to define properties of the fields.
    索引模式，用来访问Elasticsearch数据。
    ![Create index pattern](https://www.elastic.co/guide/en/kibana/7.17/management/index-patterns/images/create-index-pattern.png)
    `名称`：使用可以匹配期望索引名称的即可。当创建名称的时候，会在右侧显示匹配项。可以使用`*`通配符。
    `时间戳字段`：这里选择的是`@timestamp`。
4. Kibnan Discover
    [Discoveredit](https://www.elastic.co/guide/en/kibana/7.17/discover.html)
    With `Discover`, you can quickly gain insight to your data: search and filter your data, get information about the structure of the fields, and present your findings in a visualization. 
5. Kibnan Discover
    使用`Discover`，可以快速获查询和过滤数据。
    ![A view of the Discover app](https://www.elastic.co/guide/en/kibana/7.17/images/discover.png)
    > 注：有时候会出现找不到可用字段的情况，请查询一下当前的查询时间和系统时间是否一致。


