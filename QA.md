[TOC]

---

1. 无法启动怎么办？
     1. 问题
        发现启动失败，原因是之前不小心删除过ES日志目录`/var/log/elasticsearch`。
        启动日志：
        ``` shell
        [root@elk-101 ~]# systemctl start elasticsearch.service
        Job for elasticsearch.service failed because the control process exited with error code. See "systemctl status elasticsearch.service" and "journalctl -xe" for details.
        [root@elk-101 ~]# systemctl status elasticsearch.service
        ● elasticsearch.service - Elasticsearch
        Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: disabled)
        Active: failed (Result: exit-code) since Sun 2024-06-16 05:24:23 EDT; 12s ago
            Docs: https://www.elastic.co
        Process: 1581 ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid --quiet (code=exited, status=1/FAILURE)
        Main PID: 1581 (code=exited, status=1/FAILURE)

        Jun 16 05:24:23 elk-101 systemd-entrypoint[1581]: Error: A fatal exception has occurred. Program will exit.
        Jun 16 05:24:23 elk-101 systemd-entrypoint[1581]: at org.elasticsearch.tools.launchers.JvmOption.flagsFinal(JvmOption.java:119)
        Jun 16 05:24:23 elk-101 systemd-entrypoint[1581]: at org.elasticsearch.tools.launchers.JvmOption.findFinalOptions(JvmOption.java:81)
        Jun 16 05:24:23 elk-101 systemd-entrypoint[1581]: at org.elasticsearch.tools.launchers.JvmErgonomics.choose(JvmErgonomics.java:38)
        Jun 16 05:24:23 elk-101 systemd-entrypoint[1581]: at org.elasticsearch.tools.launchers.JvmOptionsParser.jvmOptions(JvmOptionsParser.java:135)
        Jun 16 05:24:23 elk-101 systemd-entrypoint[1581]: at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:86)
        Jun 16 05:24:23 elk-101 systemd[1]: elasticsearch.service: main process exited, code=exited, status=1/FAILURE
        Jun 16 05:24:23 elk-101 systemd[1]: Failed to start Elasticsearch.
        Jun 16 05:24:23 elk-101 systemd[1]: Unit elasticsearch.service entered failed state.
        Jun 16 05:24:23 elk-101 systemd[1]: elasticsearch.service failed.
        ```
        执行`journalctl -xe`,
        ``` shell
        [root@elk-101 ~]# journalctl -xe
        --
        -- A new session with the ID 65 has been created for the user root.
        --
        -- The leading process of the session is 4143.
        Jun 16 02:37:07 elk-101 sshd[4143]: pam_lastlog(sshd:session): unable to open /var/log/lastlog: No such file or directory
        Jun 16 02:37:07 elk-101 sshd[4151]: lastlog_openseek: Couldn't stat /var/log/lastlog: No such file or directory
        Jun 16 02:37:07 elk-101 sshd[4151]: lastlog_openseek: Couldn't stat /var/log/lastlog: No such file or directory
        Jun 16 02:37:07 elk-101 systemd[1]: Started Session 66 of user root.
        -- Subject: Unit session-66.scope has finished start-up
        -- Defined-By: systemd
        -- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
        --
        -- Unit session-66.scope has finished starting up.
        --
        -- The start-up result is done.
        Jun 16 02:37:07 elk-101 sshd[4147]: pam_unix(sshd:session): session opened for user root by (uid=0)
        Jun 16 02:37:07 elk-101 systemd-logind[615]: New session 66 of user root.
        -- Subject: A new session 66 has been created for user root
        -- Defined-By: systemd
        -- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
        -- Documentation: http://www.freedesktop.org/wiki/Software/systemd/multiseat
        --
        -- A new session with the ID 66 has been created for the user root.
        --
        -- The leading process of the session is 4147.
        Jun 16 02:37:07 elk-101 sshd[4147]: pam_lastlog(sshd:session): unable to open /var/log/lastlog: No such file or directory
        Jun 16 02:38:04 elk-101 polkitd[614]: Registered Authentication Agent for unix-process:4177:22180078 (system bus name :1.211 [/usr/bin/pkttyagent --notify-fd 5 --fallback], object path /org/freedesktop
        Jun 16 02:38:04 elk-101 systemd[1]: Starting Elasticsearch...
        -- Subject: Unit elasticsearch.service has begun start-up
        -- Defined-By: systemd
        -- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
        --
        -- Unit elasticsearch.service has begun starting up.
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: Exception in thread "main" java.lang.RuntimeException: starting java failed with [1]
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: output:
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: [0.001s][error][logging] Error opening log file '/var/log/elasticsearch/gc.log': No such file or directory
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: [0.001s][error][logging] Initialization of output 'file=/var/log/elasticsearch/gc.log' using options 'filecount=32,filesize=64m' failed.
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: error:
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: Invalid -Xlog option '-Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m', see error log fo
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: Error: Could not create the Java Virtual Machine.
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: Error: A fatal exception has occurred. Program will exit.
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: at org.elasticsearch.tools.launchers.JvmOption.flagsFinal(JvmOption.java:119)
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: at org.elasticsearch.tools.launchers.JvmOption.findFinalOptions(JvmOption.java:81)
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: at org.elasticsearch.tools.launchers.JvmErgonomics.choose(JvmErgonomics.java:38)
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: at org.elasticsearch.tools.launchers.JvmOptionsParser.jvmOptions(JvmOptionsParser.java:135)
        Jun 16 02:38:08 elk-101 systemd-entrypoint[4183]: at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:86)
        Jun 16 02:38:08 elk-101 systemd[1]: elasticsearch.service: main process exited, code=exited, status=1/FAILURE
        Jun 16 02:38:08 elk-101 systemd[1]: Failed to start Elasticsearch.
        -- Subject: Unit elasticsearch.service has failed
        -- Defined-By: systemd
        -- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
        --
        -- Unit elasticsearch.service has failed.
        --
        -- The result is failed.
        Jun 16 02:38:08 elk-101 systemd[1]: Unit elasticsearch.service entered failed state.
        Jun 16 02:38:08 elk-101 systemd[1]: elasticsearch.service failed.
        Jun 16 02:38:08 elk-101 polkitd[614]: Unregistered Authentication Agent for unix-process:4177:22180078 (system bus name :1.211, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale en_US
        ```
    2. 分析 & 解决
        执行`systemctl cat elasticsearch.service`命令，发现
        查看ES的配置文件`/etc/elasticsearch/elasticsearch.yml`，发现日志目录确实是`/var/log/elasticsearch`。
        ``` yml
        # ----------------------------------- Paths ------------------------------------
        #
        # Path to directory where to store the data (separate multiple locations by comma):
        #
        path.data: /var/lib/elasticsearch
        #
        # Path to log files:
        #
        path.logs: /var/log/elasticsearch
        ```
        解决方案是，重建logs目录。注意修改文件权限，从其它节点获知目录节点的权限是:
        ``` shell
        [root@elk-102 ~]# ll /var/log/ | grep elasticsearch
        drwxr-s---. 2 elasticsearch elasticsearch    8192 Jun 16 00:00 elasticsearch
        ```
        解决步骤：
        ``` shell
        # 重建目录
        mkdir -p /var/log/elasticsearch
        # 重置文件夹访问模式
        chmod 700 /var/log/elasticsearch
        chmod g+s /var/log/elasticsearch
        # 重置文件（夹）
        chown -R elasticsearch:elasticsearch /var/log/elasticsearch
        ```
        重新启动即可

2. 
