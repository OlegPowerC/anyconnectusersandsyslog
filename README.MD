RU: Тестовая третья версия. Статистика пользователей AnyConnect + UI с базой на Postgre SQL + Syslog server + Syslog UI

Установка (на докер сервер):<br>
git clone https://github.com/OlegPowerC/anyconnectusersandsyslog.git<br>
Зайти в папку и запустить файл initdb.sh<br>
<br>
cd anyconnectusersandsyslog<br>
chmod +x initdb.sh<br>
./initdb.sh<br>
<br>
Дождаться готовности PSQL сервера<br>
Не должно быть сообщений об ошибках и вывод должен быть примерно следующий:<br>
server started<br>
<br>
/usr/local/bin/docker-entrypoint.sh: sourcing /docker-entrypoint-initdb.d/1_createdb.sh<br>
CREATE ROLE<br>
CREATE DATABASE<br>
GRANT<br>
<br>
/usr/local/bin/docker-entrypoint.sh: sourcing /docker-entrypoint-initdb.d/2_createtabledb.sh<br>
CREATE TABLE<br>
CREATE TABLE<br>
CREATE TABLE<br>
CREATE TABLE<br>
CREATE FUNCTION<br>
CREATE FUNCTION<br>
CREATE FUNCTION<br>
CREATE FUNCTION<br>
CREATE INDEX<br>
CREATE TRIGGER<br>
CREATE TRIGGER<br>
CREATE TRIGGER<br>
CREATE TRIGGER<br>
ALTER TABLE<br>
ALTER TABLE<br>
...........<br>
PostgreSQL init process complete; ready for start up.<br>
...........<br>
...........<br>
LOG:  database system is ready to accept connections<br>
<br>
После этого остановить запущеный контейнер - ctrl+c<br>
<br>
Затем можно запускать все контейнеры: docker-compose up<br>
<br>
По умолчанию интерфейс будет доступен по адресу HTTP://<ваш docker сервер>:8182<br>
SQL порт TCP снаружи 5442 (можно выключить) Порт TCP 7002 слушает Call-Home сообщения от Cisco ASA<br>
<br>
Со стороны Cisco ASA:<br>
<br>
logging list SSLUSERS2 level informational<br>
logging list SSLUSERS2 message 605005<br>
logging list SSLUSERS2 message 611103<br>
logging list SSLUSERS2 message 113039<br>
logging list SSLUSERS2 message 113019<br>
logging list SSLUSERS2 message 722051<br>
logging trap SSLUSERS2<br>
logging host inside<br>
<br>
event manager applet StartSession<br>
&nbsp;event syslog id 722051 occurs 1<br>
&nbsp;action 0 cli command "call-home send alert-group snapshot profile VPNSH"<br>
&nbsp;output none<br>
<br>
event manager applet EndSession<br>
&nbsp;event syslog id 113019 occurs 1<br>
&nbsp;action 0 cli command "call-home send alert-group snapshot profile VPNSH"<br>
&nbsp;output none<br>
<br>
service call-home<br>
call-home<br>
&nbsp;alert-group-config snapshot<br>
&nbsp;&nbsp;add-command "show vpn-sessiondb anyconnect"<br>
&nbsp;profile VPNSH<br>
&nbsp;&nbsp;destination transport-method http destination address http http://:7002/vpnsession msg-format xml destination transport-method http subscribe-to-alert-group snapshot periodic interval 1440<br>
<br>
Для геолокации по IP адресу используется сервис https://ipstack.com/
Необходимо получить API ключ и указать его в docker-compose.yml
параметер GEOLOCATIONAPIKEY:

Добавлен Syslog сервер доступный на порту 8081

Для Syslog коллектора, добавлена возможность фильтрации сообщений от Cisco ASA, либо конкретными классами (первые 3 цифры в заголовке сообщения типа: %ASA-6-302021 вот 302 и будет класс, 
либо конкретным сообщением например 302021 а так же диапазонами например 302021-302024, разделять можно запятыми)
Параметр ASAIGNOREMSG:
Пример ASAIGNOREMSG: 305,302021-302024

Так же следует во всех контейнерах выставить одно и то же время (часовой пояс):
TZ: Europe/Moscow
А для Postgres еще и
PGTZ: Europe/Moscow

Europe/Moscow для примера.

===================================================================================================================
EN: Betta version 3. AnyConnect users statistics + UI common Postgre SQL database + Syslog server + Syslog UI

Install (on docker host): git clone https://github.com/OlegPowerC/anyconnectusersandsyslog.git

Run initdb.sh for create database and tables

cd anyconnectusersandsyslog
chmod +x initdb.sh
./initdb.sh

Wait for PSQL complete initial work and start, 
log must be like:
server started<br>
<br>
/usr/local/bin/docker-entrypoint.sh: sourcing /docker-entrypoint-initdb.d/1_createdb.sh<br>
CREATE ROLE<br>
CREATE DATABASE<br>
GRANT<br>
<br>
/usr/local/bin/docker-entrypoint.sh: sourcing /docker-entrypoint-initdb.d/2_createtabledb.sh<br>
CREATE TABLE<br>
CREATE TABLE<br>
CREATE TABLE<br>
CREATE TABLE<br>
CREATE FUNCTION<br>
CREATE FUNCTION<br>
CREATE FUNCTION<br>
CREATE FUNCTION<br>
CREATE INDEX<br>
CREATE TRIGGER<br>
CREATE TRIGGER<br>
CREATE TRIGGER<br>
CREATE TRIGGER<br>
ALTER TABLE<br>
ALTER TABLE<br>
...........<br>
PostgreSQL init process complete; ready for start up.<br>
...........<br>
...........<br>
LOG:  database system is ready to accept connections<br>
<br>
then exit - ctrl+c

Now You can start all containers: docker-compose up

By default: HTTP://<Your docker host ip>:8182 AnyConnect Users UI SQL server accessible by port TCP 5442 (You can disable it) Port TCP 7002 accept Call-Home message from Cisco ASA

On Cisco ASA You must configure next:

<br>
logging list SSLUSERS2 level informational<br>
logging list SSLUSERS2 message 605005<br>
logging list SSLUSERS2 message 611103<br>
logging list SSLUSERS2 message 113039<br>
logging list SSLUSERS2 message 113019<br>
logging list SSLUSERS2 message 722051<br>
logging trap SSLUSERS2<br>
logging host inside<br>
<br>
event manager applet StartSession<br>
&nbsp;event syslog id 722051 occurs 1<br>
&nbsp;action 0 cli command "call-home send alert-group snapshot profile VPNSH"<br>
&nbsp;output none<br>
<br>
event manager applet EndSession<br>
&nbsp;event syslog id 113019 occurs 1<br>
&nbsp;action 0 cli command "call-home send alert-group snapshot profile VPNSH"<br>
&nbsp;output none<br>
<br>
service call-home<br>
call-home<br>
&nbsp;alert-group-config snapshot<br>
&nbsp;&nbsp;add-command "show vpn-sessiondb anyconnect"<br>
&nbsp;profile VPNSH<br>
&nbsp;&nbsp;destination transport-method http destination address http http://:7002/vpnsession msg-format xml destination transport-method http subscribe-to-alert-group snapshot periodic interval 1440<br>
<br>

Gelolocation data get by https://ipstack.com/
You must get API key and provide it ib: docker-compose.yml
parameter - GEOLOCATIONAPIKEY:

Now added Syslog server and GUI.
Accesible on 8081 port

You can filter unwanted message from Cisco ASA use parameter:
ASAIGNOREMSG:
Provide class of the message, or full message id (first 3 digit in message id is class: %ASA-6-302021 class is 302,
full message id is 302021, You can provide range of id's like: 302021-302024 and provide many messages and/or clases divided by commas)
For example ASAIGNOREMSG: 305,302021-302024

You must set timezone for all containers:
TZ: Europe/Moscow
For Postgress also set
PGTZ: Europe/Moscow

Europe/Moscow for example.
