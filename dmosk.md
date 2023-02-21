

В репозиториях CentOS и Ubuntu, по умолчанию, нет пакета для установки Grafana. 

Первым делом будет установлен сам репозиторий, а после — нужный нам пакет. 

Также, в качестве примера, мы подключим Grafana к системе мониторинга Zabbix и построим график для метрики утилизации процессора.


Метод установки, описанный ниже позволит поставить последнюю версию графаны, которая доступна в репозитории. 

Как правило, это то, что нужно в большинстве случаев.

Но если нам необходимо установить конкретный релиз программы, переходим на официальную страницу загрузки Grafana, 

выбираем желаемую версию и следуем инструкции для соответствующей операционной системы.

https://grafana.com/grafana/download

### Установка и запуск на CentOS / Red Hat

#### Создаем файл конфигурации репозитория для графаны:
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

#### Теперь можно устанавливать: и отвечаем Y на все запросы.
```
yum install grafana
```

#### Настройка брандмауэра
#### По умолчанию, Grafana работает на порту 3000. Для возможности подключиться к серверу открываем данный порт в фаерволе:
```
firewall-cmd --permanent --add-port=3000/tcp
firewall-cmd --reload
```

### Запуск сервиса

#### Разрешаем автозапуск:
```
systemctl enable grafana-server
```

#### Запускаем:
```
systemctl start grafana-server
```


### Установка и запуск на Ubuntu / Debian

#### Добавляем репозиторий командой:
```
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

#### Устанавливаем ключ для проверки подлинности репозитория графаны:
```
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
```

#### Обновляем список портов:
```
apt-get update
```
#### Выполняем установку: ... и отвечаем Y на запрос.
```
apt-get install grafana
```

#### Настройка брандмауэра По умолчанию, в Ubuntu брандмауэр не блокирует соединения. 
#### Но если в нашей системе он используется, необходимо добавить порт одной из команд:

#### при использовании iptables:
#### если при вводе второй команды система выдаст ошибку, устанавливаем необходимый пакет командой apt-get install iptables-persistent.
```
iptables -A INPUT -p tcp --dport 3000 -j ACCEPT
netfilter-persistent save

```
#### при использовании ufw:
```
ufw allow 3000/tcp
ufw reload
```

#### Разрешаем автозапуск:
```
systemctl enable grafana-server
```

#### Запускаем
```
systemctl start grafana-server
```

#### Адрес grafana
```
http://<IP-адрес сервера>:3000. 
```

#### Для авторизации используем логин и пароль
```
admin / admin
```
 
#### Добавляем плагин для работы с Zabbix Установка плагина для подключения к Zabbix выполняется командой:
```
grafana-cli plugins install alexanderzobnin-zabbix-app
```

#### После окончания установки мы должны увидеть:
Installed alexanderzobnin-zabbix-app successfully

#### Перезагружаем сервер графаны:
```
systemctl restart grafana-server
``` 

Переходим к веб-интерфейсу и открываем управление плагинами:

В открывшемся списке находим Zabbix и переходим к нему:

Активируем его, кликнув по Enable:

Добавляем источник данных

Переходим в раздел Configuration - Data Sources:

Кликаем по Add data source:

Выбираем Zabbix в качестве источника данных:

На открывшейся странице в разделе «HTTP», поле URL вводим http://<путь до zabbix>/api_jsonrpc.php, например:

Ниже, в разделе «Zabbix API details», вводим логин и пароль для учетной записи с правами выполнения запросов API, 
а также выбираем версию нашего сервера Zabbix:


по умолчанию, в Zabbix создается учетная запись с правами администратора Admin с паролем zabbix. 

Однако, эту запись лучше использовать для проверки, а для целей интеграции лучше создать нового пользователя.

Нажимаем на Save & Test. Готово.


Создаем график на основе метрики в Zabbix

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


#### Настройка сервера Grafana Открываем на редактирование конфигурационный файл:
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

### Анонимный доступ

При необходимости, мы можем дать беспарольный доступ к системе.

#### Для этого открываем конфигурационный файл:
```
vi /etc/grafana/grafana.ini

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


















