### 2.2 Inventory
Структура конфигурационных файлов Ansible, которая называется Inventory:
 Подключение к машинам осуществляется через протокол ssh(linux) and powershell remoting(windown), agent в машинах не используются
 Информация о целевых системах располагается в inventory(/etc/ansible/hosts)

![[inventory.png]]

#### 2.6 Playbooks
Playbooks работает вместе с Inventory, оттуда он забирает информацию о машинах.
command, script, yum, service - это модули ansible
Запустить Playbook командой: 
```
ansible-playbook "playbook filename" 
```
и
```
ansible-playbook "playbook filename" -i inventory
```
при добавлении опции -v(можно указать разное кол-во, чем больше, тем подробнее будет информация)


#### 2.6 Modules
```
service: name=postgresql state=started
service:
    name: postgresql
    state: started
```
started(а не start) - это идемпотентность(Одиноковый результат при повторном применении операции к объекту)

script: /home/user/test.sh arg1 arg2

command: cat /etc/resolv.conf(cat resolv.conf chdir=/etc)

lineinfile - ищет линию в файле и заменяет ее, либо добавляет, если ее ранее не было.
```
tasks:
    - lineinfile:
        path: /etc/resolve.conf
        line: 'nameserver 10.1.250.10'
```

```
-
    name: 'Execute a script on all web server nodes and start httpd service'
    hosts: web_nodes
    tasks:
        -
            name: 'Update entry into /etc/resolv.conf'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver 10.1.250.10'

        -
            user:
                uid: 1040
                group: developers
        -
            name: 'Execute a script'
            script: /tmp/install_script.sh
        -
            name: 'Start httpd service'
            service:
                name: httpd
                state: present
```

#### 2.12 Variables

![[var1.png]]
![[variables.png]]

## 3. Механика Ansible
#### 3.1 Conditionals (Условные выражения)
![[3-1.png]]

![[3-2.png]]
![[3-3.png]]

![[3-4.png]]
![[3-5.png]]
![[3-6.png]]

#### 3.2 Loops

![[3-2-1.png]]
![[3-2-2.png]]
![[3-2-3.png]]
Оба плейбука дадут одинаковые результаты, большой разницы в них нет

![[3-2-4.png]]
Это несколько вариантов пользования with_* , то что идет после with_ явл-ся плагином. 
Директива with_ не устарела, но рекомендуют заменять ее на Loops + filter где это возможно. Но есть with_, которые заменить не получится(with_file,with_url,with_env,with_ini,with_pipe)

#### 3.3 Roles
##### Include:

![[3-6-1.png]]

##### Task и Vars с Include:
![[3-6-2.png]]

![[3-6-3.png]]

##### Roles:

![[3-6-4.png]]
Есть сообщество ansible-Galaxy, хранилище полезных плейбуков. Лучше посмотреть и познакомиться!

3-6-5 и 3-6-6 методом подтягивания архитектуры ansible и далее распределения ролей

![[3-6-5.png]]
![[3-6-6.png]]
Есть дефолтное место хранения ролей /etc/ansible/roles, этот путь определен в конфиге /etc/ansible/ansible.cfg в строке roles_path(при необходимости можно путь изменить)

поиск необходимой роли:
![[3-6-7.png]]

для использования этой роли необходимо выполнить install с именем роли:
![[3-6-8.png]]

использования дополнительных функций:
![[3-6-9.png]]

Чтобы посмотреть какие роли установлены использовать команду
```
ansible-galaxy list
```

Для просмотра места, где будут установлены роли
```
ansible-config dump | grep ROLE
```

А для того, чтобы установить роли в текущую папку, нужно использовать команду
```
ansible-galaxy install "name_role" -p ./roles
```
