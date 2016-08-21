---
layout: post
title: "使用ansible做自动化部署"
description: ""
category: "Software Engineering"
tags: [ansible]
---
{% include JB/setup %}

## 背景
很早之前就听说过ansible这个工具，但是一直没有真正的去用它，最近由于公司要将相关应用的生产环境迁移到aws上，中间再部署一些环境的时候感觉非常麻烦，所以想一种更好的方式来做自动化的部署，这样即使以后再有类似的迁移，也会轻松一些

## 安装

安装的方式有很多种，基本上属于开箱即用的，例如：

- Ubuntu OS

```
sudo apt-get install ansible
```

- Centos OS

```
sudd  yum install ansible
```

- 使用pip

```
pip install ansible
```



## ansible中的概念

ansible的使用有两种方式
  
  - 使用ad-hoc，可以直接在直接使用ansible执行一些命令/module
  - 使用playbook的方式来组织，像写代码一样即把你的部署工作流写在文件里面，可以做一些很好的抽象，也有利于使用版本控制工具来管理的你的部署过程

### ad-hoc

根据[官方定义](http://docs.ansible.com/ansible/intro_adhoc.html)
：

>An ad-hoc command is something that you might type in to do something really quick, but don’t want to save for later.

我的理解是，就是你可以直接在像使用使用shell命令一样来使用，就是一次性命令，比如：

```
ansible -i inventory all -m ping

ansible -i inventory webserver -m shell -a "echo 'Ansible Ad-Hoc Test' "
```

这个命令的意思是说对inventory中所有定义的host做ping操作，比如，我的inventory可以这样定义：

```
# inventory

[webserver]
192.168.1.2

[dbserver]
192.168.1.3
```

使用这种方式的优点是可以快速的实验，但是缺点是组织起来不够方便，当然也可以将这些过程写在shell脚本中，但是显然，这并不是ansible官方推荐的一种方式。我自己习惯上是使用一些最佳实践的方式去做。

### playbook

相比之下，用playbook的方式就会有很多的优点，比如：我们可以按照role的方式来组织我们的部署任务，每个role又可以用一种模块化的方式来组织task， 典型的一个playbook类似于：

```
- name: Test your playbook
  hosts: dbserver
  vars:
    db_user: db
    db_password: db
    db: db
    
  pre_tasks:
    - name: You can defined some pre-tasks
      debug: msg="This is a pre task"
      
  tasks:
    - name: You can defined some tasks
      command: echo "This is task1"
      register: 
        task1_exe_status
      
    - name: run task2 when task1 run successfully
      command: echo "This is task2"
      when: task1_exe_status.rc == 0
      notify:
        - execute a handler task
      tags:
        - db
        
  roles:
    - {role: "ansible-mysql"}
     
  handlers:
    - name: execute a handler task
      debug: msg="This is a handler task"
  
```
#### Variable
如上面所例子，我们可以在playbook中定义变量，相比一些hardcode，使用变量可能是一种更灵活的方式

变量的定义可以在多个地方，
  - inventory
  - role中的defaults/中或者vars/中
  - playbook
  - 也可以从命令行中直接传一个变量
  
具体，变量又有很多类型，比如list, dict等，具体可以参考文档: [Variables](http://docs.ansible.com/ansible/playbooks_variables.html)

#### Role

如上面例子， `ansible-mysql`就是一个role，在这个role中，我们可以定义我们在部署ansible-mysql时的整个过程，比如：

- 根据不同的操作系统， 安装不同的软件源
- 执行相关的命令安装某个版本的Mysql
- 配置MySQL的相关文件，比如开机启动项、数据文件路径、日志等
- ...

一个典型的role包括：

```
playbook.yml
roles/
  ansible-mysql/
    defaults/
    vars/
    tasks/
    handlers/
    files/
    templates/
    meta/
```

- defaults: 里面会定义一些默认变量，比如

```
# defaults/main.yml
# defaults file for ansible-mysql

db_user: db
db_password: db
db: db
db_host: localhost

# ...

```
defaults中定义的变量可以在外部任意地方覆盖

- vars: 里面定义一些不会改变的变量（但是是否能够在外部覆盖呢？TODO）

- tasks: 里面会定义具体的任务
-  handlers: 定义改role需要的handler
-  files: 定义一些内容不会改变的配置文件
-  templates: 可以定义一些使用Jiaja2定义的模板
-  meta: 定义作者信息以及该role的信息、版权等




#### Filter

通过使用filter，我们可以动态的更改变量内容，比如：

```
# some filter example 

- name: omit some args if one variable not defined
  wait_for: port=80 host={{db_host|default(omit))}}
  
- name: use default value if one variable not defined
  wait_for: port=80 host={{db_port|default(5433)}}
  
```

## 一些Tips

- 怎样定义系统的环境变量？

可以使用environment定义变量：

```
 - debug: define some environment
   environment:
     MYSQL_HOME: /opt/mysql
     MYSQL_DATA_PATH: /opt/mysql/data
```

- 任务失败后怎样retry任务？

可以使用until来重试任务

```

 name: retry task
  shell: cat /var/log/mysql.log
  register: result
  until: result.stdout.find('Successfully init')
  retries: 5
  delay: 10    
```

- 注意handler执行的顺序和生命周期

    - role中定义的hanlder在role执行结束后将会被flush
    - handler是按照定义的顺序来执行的，**不是按照notify的顺序来执行的**
    
    
- 注意sudoer文件中secure_path变量的定义

在切换到sudo的时候，默认只加载secure_path中定义的环境变量，在Amazon Linux中的变量值为:/usr/bin:/usr/sbin:/sbin:/bin，这就导致/usr/local/bin中的可执行文件不能被找到，所以要注意这一点


- 使用动态的inventory

在AWS EC2的实例中的公网IP会随着实例的生命周期结束后而改变，所以在一些云服务中公网IP是动态的，但是ansible官网提供了一个ec2.py脚本，可用于根据某些规则动态获取实例的公网IP，比如

```
ansible-playbook -i ec2.py deploy.yml tag_key_value
```

- 尽量使用module的方式来执行任务，避免使用shell

比如，如果我们想安装一个ansible，我们可以

```
- name: use shell
  shell: yum install ansible -y
  
- name: use module
  yum: name=ansible state=present
  

- name: use generatic way
  package: name=ansible state=present
``` 

- 使用可以rollback的方式来处理templates文件

默认的，ansible会把template文件处理后copy到远程定义的位置，但是当配置出现问题的时候，这样不利于我们rollback配置文件，我们可以先部署整个项目代码，然后在从远程机器上拉取current/中的template配置文件到本地，然后再在本地处理这个template文件到远程服务器，rollback时依然采取这种方式





