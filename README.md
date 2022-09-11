WSERVER, 

## B1 Создание Инфраструктуры
### B1.M1 тех документация не запрашивалась

Узнаём запрашивалась ли тех. документация

### B1.M2 Имена устройств соответствуют схеме сети

Проверяем на трёх устройствах с разной ОС

**Windows**: `get-computerinfo -property csname` - OHIOWSERVER	ohioad

**Linux**: `hostnamectl status` ATHLANTALSRV	athlappsrv

Смотрим `static name`

**vyos**: OHIOSW	ohioswitch

```shell
configure //если еще не в режиме конфига
show system host-name
```

В vyos должны увидеть обязательно

```shell
host-name ...
```

Если видим перед значением выше `>` - конфиг не применён - значит аспект не выполнен

**OPNsense**: `hostname -s` ATHLANTAFW	athfirewall

**Аспект + если все проверки успешны**

### B1.M3 DNS-имена заданы верно

Проверяем на трёх устройствах с разной ОС

**Windows**: `get-computerinfo -property csdomain` OHIOWSERVER osng.local

через графу на WSERVER: app+web+www -> one IP (or cname), athfw -> ip, ohiosw -> ip

**Аспект + если все проверки успешны**

### B1.M4 Все устройства доступны по DNS именам (проверка если 19)

WSERVER -> ping (app.asng.local, athfw.osng.local, **ohiosw.osng.local**, **ohiofw.osng.local**)

Если 19 + -> app.asng.local, athfw.osng.local, **ohiosw.osng.local**, **ohiofw.osng.local**

Если 19 - -> **ohiosw.osng.local**, **ohiofw.osng.local**

На трёх устройствах с разной ОС пингуем самих себя по FQDN

**Аспект + если все 3 проверки успешны**

### B1.M5 Названия сетей заданы верно

**vyos**: `show interfaces ethernet`

OHIOSW -> OWAN_NET, OSRV_NET, OCLI_NET

ATHLANTASW -> AFW_NET, ASRV_NET, ACLI_NET

ищем в выводе `description` с верным значением

### B1.M6 На athfw заданы верные адреса

**AFW**

LAN -> 10.10.10.1/30

OPT1 (gre) -> 1.1.1.2/30

WAN -> 178.207.179.6/29


**Аспект + если все проверки успешны**

### B1.M7 на ath-switch заданы верные адреса

**vyos**: `show interfaces ethernet`

eth0 -> 10.10.0.2/30

eth1 -> 192.168.11.1/26

eth2 -> 192.168.12.1/24

### B1.M8 на OHIO-switch заданы верные адреса

**vyos**: `show interfaces ethernet`


eth0 -> 10.10.10.2/30

eth1 -> 172.16.1.1/26

eth2 -> 172.16.2.1/24

**Аспект + если совпадают ВСЕ параметры**

### B1.M9 на OHIO-FW заданы верные адреса

**OPNsense**: `ifconfig` 

LAN -> 10.10.0.1/30

OPT1(gre) -> 1.1.1.1/30

WAN -> 77.34.141.141/22

**Аспект + если совпадают ВСЕ параметры**

### B1.M10 На OHIO-WSERVER разверут DNS сервер

На WSERVER -> SM -> DNS, если ОК -> ОК

### B1.M11 На DNS-сервере настроен форвардинг на 8.8.8.8

**Windows**: `Get-DnsServerForwarder`

В выводе должны найти 8.8.8.8

### B1.M12 На WSERVER разверут DHCP сервер  Pool 1 верный

На OHIO-WSERVER

**Windows**

Смотрим графику:

- pool 172.16.2.0 255.255.255.0 OHIO_CLI 25-100
- router 172.16.2.1
- dns 172.16.1.7
- DN - osng.local

Ищем что по заданию

**Аспект + если совпадают ВСЕ параметры**

### B1.M13 На ATHLANTA-SW разверут DHCP сервер  Pool 2 верный

На ATHLANTA-SW

```
show service dhcp-server
```

- name ATHLANTA
- subnet 192.168.12.0/24
- routeer 192.168.12.1
- dns 172.16.1.7
- dn - osng.loca
- range - 192.168.12.13-33

Ищем что по заданию

**Аспект + если совпадают ВСЕ параметры**

### B1.M14 настроен dhcp-relay на OHIO-SW


**VyOS**

```
show service dhcp-relay
```

- eth1
- eth2
- 172.16.1.7

**Важно:** если перед выводом `+` - конфиг не применён и аспект не засчитывается

**Аспект + если совпадают ВСЕ параметры**

### B1.M15 ath-wpc получает верный адрес по DHCP

**Win**: Get-NetIPadress

- address 192.168.12.13-33
- prefix - dhcp

**Аспект + если совпадают ВСЕ параметры**


### B1.M16 ohio-wincli получает верный адрес по DHCP

**Win**: Get-NetIPadress

- address 172.16.2.25-100
- prefix - dhcp

**Аспект + если совпадают ВСЕ параметры**

### B1.M17 В домене создана необходимая структура

На WSERVER

Смотрим графой в users & compucters AD

domain_users ->
  - directors
    - athlanta_dir -> mikhail
    - ohio_dir -> aleksey
  - programmers
    - athlanta_programmer -> kirill
    - ohio_programmer -> anton

**Аспект + если совпадают ВСЕ параметры**

### B1.M18 Все клиенские машины включены в домен osng.local

На WSERVER

- ATHWPC
- OHIOLCLI
- OHIOWINCLI

LCLI -> 
terminal -> aleksey@osng.local

Если 2/3 -> OK

**Аспект + если совпадают ВСЕ параметры**

### B1.M19  GRE туннель между FW_OHIO: 1.1.1.1/30  и FW_ATHLANTA: 1.1.1.2/30

OHIO_FW

ifconfig -> ip на правильных интерфейс GRE

ping 1.1.1.2

ATHLANTA_FW

ifconfig -> ip на правильных интерфейс GRE

ping 1.1.1.2

Пингуем ту сторону туннеля (туннельный IP по топологии)

**Аспект + если ВСЕ проверки ОК**

### B1.M20 между всеми филиалами OSPFv2

Правильнее прочекать на любом из роутеров

OHIOSW

```
ip route
```

- 192.168.11.0/26
- 192.168.12.0/24

ATHSW

```
ip route
```

- 10.10.10.0/30
- 192.168.11.0/26
- 192.168.12.0/24

на Vyos в операционнном режиме

```
ip route
```
- 10.10.0.0/30
- 172.16.1.0/26
- 172.16.2.0/24

Ищем все нужные сети. **Важно** - удаленные сети, должны приехать по протоколу **zebra**

**Аспект + если ВСЕ проверки ОК**

### B1.M21 На ATHLANTA-LSRV на NGINX  доступен www.osng.local

С ATHLANTA-LSRV

```
curl http://www.osng.local
```

The OSN Games main site

```
curl https://www.osng.local
```

Тут видим не гору писанины, похожей на html, с траблой по сертификату

### B1.M22 ssh до всех Linux ecnhjqcnd

LSRV

ssh vyos@athsw.osng.local

ssh root@athfw.osng.local

**Аспект + если ВСЕ проверки ОК**

### B1.M23

WCLI

rdp

ohioad.osng.local

administrator:P@ssw0rd

**Аспект + если ВСЕ проверки ОК**

### B1.M24

WCLI

лезем в проводник и смотрим шару с папками юзеров

пробуем создать в макхаиль файл

**Аспект + если ВСЕ проверки ОК**

## C1 Модернизация и ввод в эксплуаацию

### Контрольный тест

`ping github.com`

Если ОК -> едем дальше

Если нет -> 0 все аспекты

### C1.M1 Nextcloud for ATHLANTA_LSRV

смотрим на ATHLANTA_LSRV наличие контейнера на базе образа nextcloud

```
docker ps -a | grep 'nextcloud'
```


### C1.M2 контейнер nextcloud_main: 

**На ATHLANTA_LSRV:**

```shell
docker ps -a | grep 'nextcloud_main'
```

долны увидеть контейнер с именем `nextcloud_main`

### C1.M3 контейнер с базой: postgres_nextcloud

**На ATHLANTA_LSRV:**

```shell
docker ps -a | grep 'postgres_nextcloud'
```

долны увидеть контейнер с именем `postgres_nextcloud`

### C1.M4 название базы: ath_nextcloud

**На ATHLANTA_LSRV (пробуем разными способами, какой сработает - так и ладно):**

```
docker exec -it postgres_nextcloud psql -U nextcloud_db -l # если сработало -> M5 да
```

```
docker exec -it postgres_nextcloud psql -U postgres -l
```

```
docker exec -it postgres_nextcloud psql -l
```

ищем базу ath_nextcloud

### C1.M5 Таймзона МСК

```
docker exec -it nextcloud_main date
```

Ищем MSK

### C1.M6 контейнер nextcloud доступен по порту 13337

**На ATHLANTA_LSRV:**

```shell
docker ps | grep 'nextcloud_main'
```

ищем запись у контейнера с nextcloud вида `0.0.0.0:13337 -> 80/TCP`

### C1.M7 контейнер nextcloud автозапуск после перезагрузки

Ребутаем ATHLANTA_LSRV

Сразу после перезагрузки смотрим оба контейнера

```shell
docker ps 
```

должно быть UP в обоих случаях

### C1.M7 необходимые группы созданы

Лезем на http://nextcloud.osng.local:13337 -> если получилось, тогда M10 тоже +

Если на 13337 не доступе, стучимся на 8080

Если ни один из вариантов не помог, тогда M7, M8, M9 и M10 - 0

входим под учеткой admin_nextcloud:SuperSecure1.

лезем в настройки -> пользователи и группы

Ищем группу

ath_user

### C1.M8 необходимые пользователи созданы

Ищем пользователей

1. ivan_slavov
  - Полное имя: Ivan Slavov
  - Mail: islavov@osng.ru
  - Группа: ath_user
2. semen_slivov
  - Полное имя: Semen Slivov
  - Mail: sslivov@osng.ru
  - Группа: ath_user

### C1.M9 Share

Смотрим доступные общие ресурсы и кому они разрешены

Должен быть каталог Share доступный всем

### C1.M10 http://nextcloud.osng.local:13337

Проверяем доступность (на самом деле уже проверили)

### C1.M11 URL панели управления: http://web.osng.local

С браузера в любой клиентской таче лезем на страницу http://web.osng.local

если увидели вход в панель значит ОК

Если панель не доступна значит - M12, M14, 

### C1.M12 Автор сервиса: admin@osng.ru

смотрим panel/docker-compose.yml (в /home/user)

### C1.M13 Внутреняя сеть 172.16.20.0/24

`docker network ls` -> ищем сеть panel
`docker network inspect <id>`

Ищем сетку 172.16.20.0/24

### C1.M14 пользователь game_admin создан, utkin_code и kuznetsov_code созданы

Настройки -> Users

### C1.M15 внутренняя сеть ноды  172.16.21.0/24

`docker network ls` -> ищем сеть wings
`docker network inspect <id>`

Ищем сетку 172.16.21.0/24

### C1.M16 alias: firstgamenode,  диапазон портов: 27000-27025

В панели Nodes -> Allocation -> IP alisas Port

### C1.M17 Сервер Minecraft_Server_1 создан

В панели: Servers -> Minecraft_Server_1

### C1.M18 Владелец сервера: game_admin

Servers -> Minecraft_Server_1 -> Owner

Ищем game_admin

### C1.M19 Port: 27013

Connection Alias - ищем 27013

### C1.M20 Ram: 2GB

Memory - 2048

### C1.M21 Disk: 4GB

Disk Space - 4096

### C1.M22 Swap: 1GB

Build Config -> Alloc Swap

### C1.M23 Nest: Minecraft

Startup Conf -> Service Config

### C1.M24 Egg: Vanilla Minecraft

Startup Conf -> Service Config
