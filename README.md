RocketMQ
=========

RocketMQ Dledger Controller 模式部署！


Focus
----------------
该模式需运行 proxy，broker 节点启动时 mqbroker 使用的 runserver.sh 管理 broker 进程，所以增加 runproxy.sh 模板，以适配 broker jvm 参数。 

Usage
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

Install this ansible role:

```shell script
ansible-galaxy install ansible-role-rocketmq
```

Playbook eg:

```yaml
---
# install: ansible-playbook rocketmq.yml --tags "namesrv"
# maintainance：ansible rmq-namesrv -m shell -a "systemctl status rocketmq-namesrv"
- hosts: rmq-namesrv
  gather_facts: True
  tags: ["default", "namesrv"]
  roles:
    - role: rocketmq
      rmq_role: "namesrv"
      rmq_role_name: "namesrv"
      rmq_controller_group_name: "cluster_name-namesrv"
      rmp_peer_hosts: "n1-ip01:9878;n2-ip02:9878;n3-ip03:9878"
      rmq_namesrv_servers: [rmq-namesrv01, rmq-namesrv02, rmq-namesrv03]

# install: ansible-playbook rocketmq.yml --tags "cluster"
# maintainance: ansible-playbook rocketmq.yml --tags "cluster" -e "action=config"
- hosts: rmq-broker01 rmq-broker02 rmq-broker03
  serial: 1
  gather_facts: True
  tags: ["cluster", "broker-a"]
  roles:
    - role: rocketmq
      rmq_role: "broker"
      broker_name: "broker-a"
      rmq_role_name: "broker-a"
      broker_listen_port: 20911
      proxy_listen_port: 28080
      proxy_grpc_port: 28081
      rmq_broker_jvm_port: 22345
      rmq_namesrv_jvm_port: 12345
      broker_cluster_name: "cluster_name"
      controller_addrs: "ip01:9878;ip02:9878;ip03:9878"
      nameserver_addrs: "ip01:9876;ip02:9876;ip03:9876"

- hosts: rmq-broker02 rmq-broker03 rmq-broker01
  serial: 1
  gather_facts: True
  tags: ["cluster", "broker-b"]
  roles:
    - role: rocketmq
      rmq_role: "broker"
      broker_name: "broker-b"
      rmq_role_name: "broker-b"
      broker_listen_port: 30911
      proxy_listen_port: 38080
      proxy_grpc_port: 38081
      rmq_broker_jvm_port: 32345
      rmq_namesrv_jvm_port: 12346
      broker_cluster_name: "cluster_name"
      controller_addrs: "ip01:9878;ip02:9878;ip03:9878"
      nameserver_addrs: "ip01:9876;ip02:9876;ip03:9876"

- hosts: rmq-broker03 rmq-broker01 rmq-broker02
  serial: 1
  gather_facts: True
  tags: ["cluster", "broker-c"]
  roles:
    - role: rocketmq
      rmq_role: "broker"
      broker_name: "broker-c"
      rmq_role_name: "broker-c"
      broker_listen_port: 40911
      proxy_listen_port: 48080
      proxy_grpc_port: 48081
      rmq_broker_jvm_port: 42345
      rmq_namesrv_jvm_port: 12347
      broker_cluster_name: "cluster_name"
      controller_addrs: "ip01:9878;ip02:9878;ip03:9878"
      nameserver_addrs: "ip01:9876;ip02:9876;ip03:9876"
```

Run playbook:

```shell script
# Install nameserver
ansible-playbook rocketmq.yml --tags "namesrv"

# Install broker
ansible-playbook rocketmq.yml --tags "cluster"

# Update config
ansible-playbook rocketmq.yml --tags "cluster" -e "action=config"
```


