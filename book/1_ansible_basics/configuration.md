## Конфигурационный файл

Настройки Ansible можно менять в конфигурационном файле.

Конфигурационный файл Ansible может хранится в разных местах (файлы перечислены в порядке уменьшения приоритетности):
* ANSIBLE_CONFIG (переменная окружения)
* ansible.cfg (в текущем каталоге)
* .ansible.cfg (в домашнем каталоге пользователя)
* /etc/ansible/ansible.cfg

Ansible ищет файл конфигурации в указанном порядке и использует первый найденный (конфигурация из разных файлов не совмещается).

В конфигурационном файле можно менять множество параметров. Мы разберем лишь несколько. Остальные параметры и их описание, можно найти в [документации](http://docs.ansible.com/ansible/intro_configuration.html).

В текущем каталоге у вас уже должен быть инвентарный файл myhosts:
```
[cisco-routers]
192.168.100.1
192.168.100.2
192.168.100.3

[cisco-switches]
192.168.100.100
```

Теперь создадим в локальном каталоге конфигурационный файл ansible.cfg:
```
[defaults]

inventory = ./myhosts
remote_user = cisco
ask_pass = True
```

Разберемся с настройками в конфигурационном файле:
* ```[defaults]``` - это секция конфигурации описывает общие параметры по умолчанию
* ```inventory = ./myhosts``` - параметр inventory позволяет указать местоположение инвентарного файла.
 * Если настроить этот параметр, то нам не придется указывать, где находится файл, каждый раз когда мы запускаем Ansible
* ```remote_user = cisco``` - от имени какого пользователя будет подключаться Ansible
* ```ask_pass = True``` - этот параметр аналогичен опции --ask-pass в командной строке. Если он выставлен в конфигурации Ansible, то уже не нужно указывать его в командной строке.

Теперь вызов ad-hoc команды будет выглядеть так:
```
$ ansible cisco-routers -m raw -a "sh ip int br"
```

То есть, нам не нужно указывать инвентарный файл, пользователя и опцию --ask-pass.


### gathering

По умолчанию, Ansible собирает факты об устройствах.

Факты - это различная информация о хостах, к которым мы подключаемся. Эти факты можно использовать в playbook и шаблонах как переменные.

Сбором фактов, по умолчанию, занимается модуль [setup](http://docs.ansible.com/ansible/setup_module.html).

Но, для сетевого оборудования, модуль setup не подходит. Нам нужно использовать другие модули для сбора фактов об оборудовании (если они есть).

И, так как поведение по умолчанию - собирать факты, нам надо это отключить.
Мы можем отключать это в конфигурационном файле Ansible или в playbook.

Для того, чтобы отключить сбор фактов в конфигурационном файле, надо добавить строку:
```yml
gathering = explicit
```

Эта строка означает, что теперь, для сбора фактов об устройстве в playbook надо явно указать, что факты надо собирать (в разделе playbook мы посмотрим как это делать).

### host_key_checking

Ещё одна настройка, которая может пригодится - host_key_checking.
Она полезна в том случае, когда в управляющего хоста Ansible вы подключаетесь первый раз к большому количеству устройств.

Если указать в конфигурационном файле ```host_key_checking=False```,  будет отключена проверка ключей, при подключении по SSH.

Чтобы проверить этот функционал, удалим сохраненные ключи для устройств Cisco, к которым мы подключаемся.
Для этого, нужно удалить ключи, которые соответствуют этим устройствам из файла ~/.ssh/known_hosts.

Теперь, если мы запустим ad-hoc команду, мы получим такой вывод:
```
$ ansible cisco-routers -m raw -a "sh ip int br"
SSH password:
192.168.100.1 | FAILED | rc=0 >>
Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.

192.168.100.2 | FAILED | rc=0 >>
Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.

192.168.100.3 | FAILED | rc=0 >>
Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.
```

Теперь добавим в конфигурационный файл параметр host_key_checking:
```
[defaults]

inventory = ./myhosts

remote_user = cisco
ask_pass = True

host_key_checking=False
```

И повторим ad-hoc команду:
```
$ ansible cisco-routers -m raw -a "sh ip int br"
```

![ad-hoc](https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/ad-hoc.png)

Обратите внимание на строки:
```
 Warning: Permanently added '192.168.100.1' (RSA) to the list of known hosts.
```

Это Ansible сам добавил ключи устройств в файл ~/.ssh/known_hosts.
При подключении в следующий раз этого сообщения уже не будет.


На этом мы завершаем знакомство с конфигурационном файлом Ansible.
Другие параметры вы можете посмотреть в документации.
И можно посмотреть на пример конфигурационного файла в [репозитории Ansible](https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg).

