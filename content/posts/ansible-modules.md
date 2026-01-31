+++
date = '2026-01-23T22:35:00+08:00'
draft = false
title = 'Ansible 常用模块'

+++
## 一、文件操作类

1. copy：复制文件到远程主机

** 场景**：分发配置文件、脚本、证书等

```yaml
- name:分发Nginx配置文件
  copy:
    scr: files/nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: www
    group: www
    mode: '0644'
    backup: yes # 修改前备份
```

scr路径相对于playbook所在目录的 files/子目录。

2. template：使用Jinja2模版渲染文件

** 场景**：动态生成配置文件（如根据变量生成不同环境的配置）

```yaml
- name: 渲染应用配置文件
  template:
    src: templates/app.conf.j2
    dest: /etc/myapp/app.conf
    owner: app
    group: app
    mode: '0600'
# templates/app.conf.j2 示例：
database_host = {{ db_host }}
port = {{ app_port | defaut(8080) }}
debug = {{ debug_mode | lower }}
```

3. file：管理文件/目录属性或创建/删除

** 场景**：创建目录、修改权限、删除文件

```yaml
- name: 创建日志目录
  file:
    path: /var/log/myapp
    state: directory
    owenr: app
    group: app
    mode: '0755'
    
- name: 删除临时文件
  file:
    path: /tmp/old_file.txt
    state: absent
    
state: 可选值    
  # directory:创建目录;
  # file: 创建空的文件;
  # link: 创建软连接;
  # absent: 删除文件或目录
mode: 必须加引号,避免YAML把0755 当成八进制数解析错误
```

4. lineinfile：修改单行配置

```yaml
- name: 开启内核转发
  lineinfile:
    path: /etc/sysctl.conf
    line: 'net.ipv4.ip_forward = 1'
    create: yes
```

5. blockinfile: 插入多行配置块

```yaml
- name: 添加 SSH 公钥
  blockinfile:
    path: /root/.ssh/authorized_keys
    block: |
      ssh-rsa AAAB3... user1@host
      ssh-rsa BBBC4... user2@host
    marker: "# {mark} ANSIBLE MANAGED BLOCK"
```

## 二、包管理（Package Management）

5. yum / apt: 安装/卸载软件包(RHEL/Debian系)

** 场景**：批量安装基础软件

```yaml
- name: 安装常用的软件
  yum:
    name:
      - vim
      - net-tools
      - wget
    state: present   # 安装， absent 卸载、latest 安装并升级到最新版本
 
 - name: 卸载
   apt:
     name: apache2
     state: absent
# 使用package模块可跨平台:
- nmae:
  package:
    name: git
    state: present
```

6. package （ansible 会自动判断系统，调用yum或apt）

```yaml
- name: 跨平台安装curl
  package:
    name: curl
    state: present
```

## 三、服务管理

7. service / systemd: 管理服务启停

 ** 场景**：重启服务、开机自启

```yaml
- name: 启动并启用nginx
  syatemd:
    name: nginx
    state: started   # started 开启，stopped 关闭，restarted 重启
    enabled: yes
   
- name: 重载防火墙
  service:
    name: firewalld
    state: reloaded
```

## 四、用户与权限

8. user：用户管理

```yaml
- name: 创建无登录权限的系统用户
  user:
    name: www
    system: yes           # 创建UID < 1000 的系统用户，可以使用uid: gid:  来设置想要的id号
    shell: /sbin/nologin  # 禁止登陆，     /bin/login 允许登录
    create_home: no
```

9. authorized_key: 部署公钥到root用户

```yaml
- name: 部署公钥到root用户
  authorized_key:
    user: root
    key: "{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"
 
 # lookup ('file'...) 从控制机读取本地文件内容，自动追加到~/.ssh/authorized_keys，支持去重。  可以实现SSH免密登录的核心方法
```

## 五、安全与网络

10. firewalld：防火墙,可代替firewall-cmd命令

```yaml
- name: 开发HTTPS端口
  firewalld:
    prot: 443/tcp
    permanent: yes    # 写入永久配置（重启不失效）
    state: enabled
    immediate: yes    # 立即生效（无需reload）
```

## 六、数据库（MySQL）

11. mysql_user: （首次设密关键）

```yaml
- name: 首次设置root密码
  mysql_user:
    name: root
    password: "{{ mysql_root_pass }}"
    login_unix_socket: /var/lib/mysql/mysql.sock
```

## 七、流程控制与调试

12. wait_for: 

```yaml
- name: 等待数据库端口就绪
  wait_for:
    host: localhost
    port: 3306
    delay: 5	# 等待几秒开始检查
    timeout: 30	# 超时时间
```

13. debug

```yaml
- name: 打印主机IP
  debug:
    msg: "This server IP is {{ ansible_default_ipv4.address }}"	
# 用于调试变量、输出信息，msg 支持Jinja2表达式
# 不改变系统状态，安全用于任何环境
```

























