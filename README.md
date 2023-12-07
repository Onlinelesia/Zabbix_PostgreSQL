# Zabbix_PostgreSQL

Задание:
Проинсталлировать на существующую VM3 сервер мониторинга Zabbix одной из последних LTS версий.
Для VM1, VM2 установить Zabbix агенты, настроить мониторинг аппаратных средств VM1, VM2, а также установленных там сервисов. Добавить проверку доступности (Healthcheck) вебсайтов на CMS, установленных в LAMP стеке (веб мониторинг).
Написать инструкцию в формате «Howto», как настраивали сервисы мониторинга и добавить в отчет.
Дополнительное задание
Настроить хранилище метрик в базе данных PostgreSQL. Настроить доступ к веб-интерфейсу по протоколу HTTPS.
Настроить на VM1 для WordPress директорию Uploads (для WordPress добавить плагин https://ru.wordpress.org/plugins/wp-file-upload/), содержимое которой будет физически располагаться на VM2, т.е. директория должна быть подключена с помощью сетевой файловой системы Samba (CIFS) или NFS.
Дополнить ваш набор документов «Howto» описанием, как настраивали взаимодействие, и добавить в отчет.

## ______________________________________________________________________________________

# Установка репозитория Zabbix

Есть инструкция с оф сайта:
https://www.zabbix.com/ru/download?zabbix=6.0&os_distribution=ubuntu&os_version=22.04&components=agent&db=&ws=

Добавить репозиторий Zabbix
```
sudo wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
sudo apt update
```

Установить Zabbix сервер, веб-интерфейс и агент
```
sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

Установка PostgreSQL
```
sudo apt install postgresql postgresql-contrib net-snmp fping nmap
```
Создать и настроить БД Postgres для Zabbix
```
sudo -u postgres psql
CREATE DATABASE zabbix;
CREATE USER zabbixuser WITH PASSWORD 'password'; 
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbixuser;
```
На хосте Zabbix сервера импортировать начальную схему и данные
```
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```
Настроить базу данных для Zabbix сервера
Отредактировать файл /etc/zabbix/zabbix_server.conf
Раскомментировать и добавить пароль
DBPassword=password

Запустить процессы Zabbix сервера и агента
```
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

В браузере открыть http://IP VM3/zabbix

В настройках создать 2 хоста для VM1 и VM2, добавить их IP, выбрать группы, шаблоны. Проверить.

## _____________________________________________________________________________________________________

# Установка Zabbix агентов на VM1 и VM2:
```
sudo apt-get install zabbix-agent
```
Настроить файл /etc/zabbix/zabbix_agentd.conf
Отредактировать параметр Server и ServerActive
```
Server=VM3_IP
ServerActive=VM3_IP
```
Перезапуск службы Zabbix агента
```
sudo service zabbix-agent restart
```
# Настройка мониторинга аппаратных средств

Войти в веб-интерфейс Zabbix на VM3
В разделе "Templates" найти и добавить шаблон "Template OS Linux" для Linux-хостов.


# Добавление Healthcheck для веб-сайтов

Добавить хостам шаблон "HTTP Service".

Создание веб-сценариев
Перейти в "Configuration" -> "Web" и создать веб-сценарий для каждого веб-сайта, указав URL и параметры проверки.
Настроить Healthcheck для веб-сайтов
В разделе "Discovery" добавить новый элемент для каждого веб-сайта и настроить его параметры.
Добавление триггеров для Healthcheck
В разделе "Triggers" добавить новые триггеры для каждого веб-сайта, используя элементы Healthcheck.














