
В репозиториях CentOS и Ubuntu, по умолчанию, нет пакета для установки Grafana. 

Первым делом будет установлен сам репозиторий, а после — нужный нам пакет. 

Также, в качестве примера, мы подключим Grafana к системе мониторинга Zabbix и построим график для метрики утилизации процессора.

## Установка и запуск на CentOS / Red Hat

#### Создаем файл конфигурации репозитория для графаны. Вариант установки последней версии, доступной в репозитории.
```
vim /etc/yum.repos.d/grafana.repo

[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

#### Устаналиваем, отвечаем Y на все запросы.
```
yum install grafana
```

#### Установка конкретного релиз программы, переходим на официальную страницу загрузки Grafana, выбираем желаемую версию и следуем инструкции для соответствующей операционной системы
```
https://grafana.com/grafana/download

wget https://dl.grafana.com/oss/release/grafana-9.3.6-1.x86_64.rpm
sudo yum install grafana-9.3.6-1.x86_64.rpm
```

#### Firewall, по умолчанию, grafana работает на порту 3000
```
firewall-cmd --permanent --add-port=3000/tcp
firewall-cmd --reload
```

#### Разрешаем автозапуск
```
systemctl enable grafana-server
```

#### Запуск
```
systemctl start grafana-server
```


## Установка и запуск на Ubuntu / Debian

#### Добавляем репозиторий
```
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

#### Устанавливаем ключ для проверки подлинности репозитория графаны
```
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
```

#### Обновляем список пакетов
```
apt-get update
```
#### Выполняем установку, отвечаем Y на запрос.
```
apt-get install grafana
```

#### Установка конкретного релиз программы, переходим на официальную страницу загрузки Grafana, выбираем желаемую версию и следуем инструкции для соответствующей операционной системы
```
https://grafana.com/grafana/download

apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_9.3.6_amd64.deb
dpkg -i grafana_9.3.6_amd64.deb
```

#### Firewall, в Ubuntu брандмауэр по умолчанию не блокирует соединения. 
#### Iptables, если при вводе второй команды система выдаст ошибку, устанавливаем необходимый пакет командой apt-get install iptables-persistent.
```
iptables -A INPUT -p tcp --dport 3000 -j ACCEPT
netfilter-persistent save

```
#### ufw:
```
ufw allow 3000/tcp
ufw reload
```

#### Разрешаем автозапуск
```
systemctl daemon-reload
systemctl enable grafana-server
```

#### Запускаем
```
systemctl start grafana-server
```

#### Адрес grafana, для авторизации используем логин и пароль
```
http://ServerIP:3000

admin / admin
```


#### Устанавливаем плагин для работы с Zabbix
```
grafana-cli plugins install alexanderzobnin-zabbix-app
```

#### Перезагружаем сервер графаны:
```
systemctl restart grafana-server
``` 

#### Активируем плагин

Переходим к веб-интерфейсу и открываем управление плагинами. В открывшемся списке находим Zabbix, активируем его, кликнув по Enable.


#### Добавляем источник данных

Переходим в раздел Configuration - Data Sources:

Кликаем по Add data source:

Выбираем Zabbix в качестве источника данных:

На открывшейся странице в разделе «HTTP», поле URL вводим http://<путь до zabbix>/api_jsonrpc.php

Ниже, в разделе «Zabbix API details», вводим логин и пароль для учетной записи с правами выполнения запросов API, а также выбираем версию нашего сервера Zabbix.

по умолчанию, в Zabbix создается учетная запись с правами администратора Admin с паролем zabbix. 

Однако, эту запись лучше использовать для проверки, а для целей интеграции лучше создать нового пользователя.

Нажимаем на Save & Test. Готово.

#### Создаем график на основе метрики в Zabbix

Переходим в раздел Create - Dashboard:

Выбираем Add Query:

Заполняем поля для получения данных с Zabbix:

* где:

Query — источник данных. Выбираем Zabbix.
Query Mode — тип данных. Оставляем Metrics.
Group — группа серверов в Zabbix. Выбираем нужную нам группу.
Host — имя сервера, для которого будем вытаскивать данные.
Application — данные для какого компонента будем собирать. В данном примере, процессора.
Item — какой именно тип информации нас интересует. На скриншоте выше выбрано время простоя процессора.

При желании, можно настроить графики в разделе Visualization:

После сохраняем данные:

В открывшемся всплывающем окне задаем имя дашборду и нажимаем Save. Готово.

### Настройка https

После установки Grafana работает по протоколу http. Для настройки https необходимо выполнить 2 задачи:

Получение сертификата.
Настройка графаны.

Получаем сертификат
Для получение сертификата можно его купить или запросить бесплатно у Let's Encrypt. 

https://www.dmosk.ru/miniinstruktions.php?mini=get-letsencrypt

Предположим, что мы получили сертификат от Let's Encrypt для узла grafana.dmosk.ru и поместили его в каталог /etc/letsencrypt/live/grafana.dmosk.ru.


#### Настройка https в grafana:
```
vim /usr/share/grafana/conf/defaults.ini

...
protocol = https
...
cert_file = /etc/letsencrypt/live/grafana.dmosk.ru/fullchain.pem
cert_key = /etc/letsencrypt/live/grafana.dmosk.ru/privkey.pem
...
```

* где protocol определяет протокол, по которому будет работать веб-интерфейс grafana; 
* cert_file — путь до открытого ключа безопасности; cert_key — до закрытого.


#### Перезапускаем сервис:
```
systemctl restart grafana-server
```

#### Пробуем перейти на веб-интерфейс графаны по доменному имени. 
```
https://grafana.dmosk.ru:3000.
```

## Анонимный доступ, при необходимости, мы можем дать беспарольный доступ к системе

#### Для этого открываем конфигурационный файл
```
vim /etc/grafana/grafana.ini

[auth.anonymous]
# enable anonymous access
enabled = true

# specify organization name that should be used for unauthenticated users
org_name = Main Org.

# specify role for unauthenticated users
org_role = Viewer

```

* в данном примере:

enabled — разрешает или запрещает анонимный доступ.
org_name — название организации, к которой разрешен анонимный доступ. Организации настраиваются в веб-интерфейсе графаны (раздел Server Admin - Orgs). По умолчанию создается имя Main Org.
org_role — уровень доступа для анонимных пользователей.



#### Перезапускаем сервис:
```
systemctl restart grafana-server
```


### Настройка Grafana для аутентификации через Active Directory

Разберем пример настройки без каких-либо дополнительных возможностей — просто авторизация через AD.

#### Открываем основной конфигурационный файл Grafana:
#### и указываем использовать аутентификацию LDAP, а также указываем путь до конфигурационного файла с настройками ldap (снимаем комментарии с данных опций и приводим их к виду):
```
vim /etc/grafana/grafana.ini

...
[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
allow_sign_up = true
...
```

* где: 

enabled — параметр, который разрешает или запрещает использование ldap; 
config_file — путь до файла, где будут храниться соответствующие настройки; 
allow_sign_up — опция, отвечающая за автоматическое создание пользователей — необходима, чтобы могли авторизоваться пользователи, которых еще нет в графане.


#### Открываем конфигурационный файл ldap.toml и приводим его к виду:
```
vim /etc/grafana/ldap.toml

[[servers]]
host = "192.168.0.15"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false
bind_dn = "DMOSK\\%s"
#bind_password = 'grafana'
search_filter = "(sAMAccountName=%s)"
search_base_dns = ["dc=dmosk,dc=local"]
```

* где:

host — имя сервера контроллера домена или IP-адрес. В данном примере указан сервер 192.168.0.15.
port — используемый порт для связи с AD.
use_ssl — использовать ли SSL. В данном примере не используем.
start_tls — использовать ли STARTTLS.
ssl_skip_verify — опция проверки сертификата на корректность. Если задано false, то проверять.
bind_dn — учетная запись, от которой будут выполняться запросы в сторону LDAP. У нее могут быть минимальные права в AD. В данном примере мы используем ту же запись, от которой заходим в систему В нашем примере работаем в домене DMOSK.
bind_password — пароль для связки с AD. Обратите внимание, что данная опция должна быть закомментирована, тогда пароль будет браться из того, что введет пользователь при входе.
search_filter — ldap фильтр для поиска учетных записей, которые будут удовлетворять определенным запросам. В моем случае, cn был заменен на sAMAccountName, так как логин в AD больше соответствует данному атрибуту.
search_base_dns — в каком контейнере нужно искать пользователей. Можно задать конкретный организационный юнит. В данном примере поиск будет идти по всему домену.


#### Перезапускаем графану:
```
systemctl restart grafana-server
```


Проверка и отладка
Открываем веб-интерфейс графаны и пробуем зайти с использованием учетной записи AD. В качестве логина указываем учетную запись без префиксов самого домена, например, dmosk. И пароль от доменной записи. Авторизация должна пройти успешно.

#### Если система выдает ошибку и не пускает нашу запись, то открываем основной конфигурационный файл: ... и задаем фильтр для лога:
```
vim /etc/grafana/grafana.ini

[log]
filters = ldap:debug

```

#### Перезапускаем графану:
```
systemctl restart grafana-server
```

#### Смотрим лог:
```
tail -f /var/log/grafana/grafana.log
```

#### Авторизация на основе групп

Теперь разберем настройку предоставления прав на основе групп AD.

#### Открываем наш конфигурационный файл для настройки ldap:
```
vim /etc/grafana/ldap.toml
```

#### В начальном файле есть секции servers.group_mappings — либо редактируем их, либо создаем новые следующего вида:
```
...

[[servers.group_mappings]]
group_dn = "cn=Enterprise Admins,cn=Users,dc=dmosk,dc=local"
org_role = "Admin"
grafana_admin = true

[[servers.group_mappings]]
group_dn = "cn=Domain Admins,cn=Users,dc=dmosk,dc=local"
org_role = "Admin"

[[servers.group_mappings]]
group_dn = "cn=Редакторы графаны,ou=Группы безопасности,dc=dmosk,dc=local"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"

```


* где group_dn — группа безопасности в ldap каталоге, org_role — роль в Grafana, на основе которой пользователю будут присвоены права, grafana_admin — права суперадминистратора. В конкретном примере мы предоставляем полные права пользователям группы «Enterprise Admins», права администратора для «Domain Admins», разрешаем вносить правки пользователям группы «Редакторы графаны» и всем остальным разрешаем только просмотр данных.

#### Чтобы настройки вступили в силу, перезапускаем сервис графаны:
```
systemctl restart grafana-server
```


### Несколько контроллеров домена

Мы рассмотрели пример добавления одного сервера, выступающего контроллером домена. Графана поддерживает возможность добавления нескольких серверов — для этого необходимо после всех настроек для первого контроллера, добавить второй:

* в данном примере сначала идут настройки для контроллера 192.168.0.15, затем для 192.168.0.16.

```
vim /etc/grafana/ldap.toml

[[servers]]
host = "192.168.0.15"
...

[servers.attributes]
...

[[servers.group_mappings]]
...

[[servers]]
host = "192.168.0.16"
...

[servers.attributes]
...

[[servers.group_mappings]]
...
```


#### Перезагружаем графану:
```
systemctl restart grafana-server
```
