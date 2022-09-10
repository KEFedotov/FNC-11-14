## B1 Создание Инфраструктуры
### B1.M1 тех документация не запрашивалась

Узнаём запрашивалась ли тех. документация

### B1.M2 Имена устройств соответствуют схеме сети

Проверяем на трёх устройствах с разной ОС

**Windows**: `get-computerinfo -property csname`

**Linux**: `hostnamectl status`

Смотрим `static name`

**vyos**: 

```shell
configure //если еще не в режиме конфига
show system host-name
```

В vyos должны увидеть обязательно

```shell
host-name ...
```

Если видим перед значением выше `>` - конфиг не применён - значит аспект не выполнен

**OPNsense**: `hostname -s`

**Аспект + если все проверки успешны**

### B1.M3 DNS-имена заданы верно

Проверяем на трёх устройствах с разной ОС

**Windows**: `get-computerinfo -property csdomain`

**Linux**: `dnsdomainname`

**vyos**: 

```shell
configure //если еще не в режиме конфига
show system domain-name
```

В vyos должны увидеть обязательно

```
domain-name ...
```

Если видим перед значением выше `+` - конфиг не применён - значит аспект не выполнен

**OPNsense**: `hostname -d`

**Аспект + если все проверки успешны**

### B1.M4 Все устройства доступны по DNS именам

На трёх устройствах с разной ОС пингуем самих себя по FQDN

**Примечание:** на vyos пинг делаем из операционного режима

**Аспект + если все 3 проверки успешны**

### B1.M5 Названия сетей заданы верно

На сетевом оборудовании

**vyos**: `show interfaces ethernet`

ищем в выводе `description` с верным значением

### B1.M6 На ath_firewall заданы верные адреса

**OPNsense**: `ifconfig`

**Аспект + если все 3 проверки успешны**

### B1.M7 на ath-switch заданы верные адреса

**vyos**: `show interfaces ethernet`

### B1.M8 на OHIO-switch заданы верные адреса

**vyos**: `show interfaces ethernet`

**Аспект + если совпадают ВСЕ параметры**

### B1.M9 на OHIO-FW заданы верные адреса

**OPNsense**: `ifconfig` 

**Аспект + если совпадают ВСЕ параметры**

### B1.M10 На OHIO-WSERVER разверут DNS сервер

На OHIO-WSERVER

**Windows**

```
Get-DnsServer Setting
```

Исчем в Computer Info имя этой тачки, а в AllIPAddress её же IP

**Аспект + если совпадают ВСЕ параметры**

### B1.M11 На DNS-сервере настроен форвардинг на 8.8.8.8

**Windows**: `Get-DnsServerForwarder`

В выводе должны найти 8.8.8.8

**Linux**: `sudo cat /etc/named/named.conf | grep -A 5 '8.8.8.8'`

Вывод не должен быть пустой

**Аспект + если вывод верный**

### B1.M12 На OHIO-WSERVER разверут DHCP сервер  Pool 1 верный

На OHIO-WSERVER

**Windows**

```
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue
```

Ищем что по заданию

**Аспект + если совпадают ВСЕ параметры**

### B1.M13 На OHIO-WSERVER разверут DHCP сервер  Pool 2 верный

На OHIO-WSERVER

**Windows**

```
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue
```

Ищем что по заданию

**Аспект + если совпадают ВСЕ параметры**

### B1.M14 настроен dhcp-relay на OHIO-SW


**VyOS**

```
show service dhcp-relay
```

Должны увидеть IP dhcp-сервера и интерфейс, смотрящий на клиентов

**Важно:** если перед выводом `+` - конфиг не применён и аспект не засчитывается

**Аспект + если совпадают ВСЕ параметры**

### B1.M15 ath-wpc получает верный адрес по DHCP

**Win**: Get-NetIPadress

Чекаем IP - должен быть в сооттветствующем диапазоне и prefix - dhcp

**Аспект + если совпадают ВСЕ параметры**


### B1.M16 ohio-wincli получает верный адрес по DHCP

**Win**: Get-NetIPadress

Чекаем IP - должен быть в сооттветствующем диапазоне и prefix - dhcp

**Аспект + если совпадают ВСЕ параметры**

### B1.M17 В домене создана необходимая структура

На OHIO-WSERVER

Смотрим графой в users & compucters AD


**Аспект + если совпадают ВСЕ параметры**

### B1.M18 Все клиенские машины включены в домен osng.local

На На OHIO-WSERVER

```
Get-DnsServerResourceRecord -ZoneName ...
```

Проверяем наличие всех тачек, которые там должны быть и обязательно TimeStamp (если он 0, значит запись создана искуственно и с тачкой скорее всего не связана)

**Аспект + если совпадают ВСЕ параметры**

### B1.M19  GRE туннель между FW_OHIO: 1.1.1.1/30  и FW_ATHLANTA: 1.1.1.2/30

На чём FW_OHIO и FW_ATHLANTA???

Если Vyos:

```
show interfaces tunnel
```

Чекаем:

- Инкапсуляцию (GRE)
- Source-address (физический IP текущей тачки)
- remote (физический IP тачки на другой стороне туннеля)
- address (туннельный IP этой тачки)

Пингуем ту сторону туннеля (туннельный IP по топологии)

**Аспект + если ВСЕ проверки ОК**

### B1.M20 между всеми филиалами OSPFv2

Правильнее прочекать на любом из роутеров

на Vyos в операционнном режиме

```
ip route
```

Ищем все нужные сети. **Важно** - удаленные сети, должны приехать по протоколу **zebra**

**Аспект + если ВСЕ проверки ОК**

### B1.M21 На ATHLANTA-LSRV на NGINX  доступен www.osng.local

С ATHLANTA-LSRV

```
curl http://www.osng.local
```

Тут видим гору писанины, похожей на html

```
curl https://www.osng.local
```

Тут видим гору писанины, похожей на html, но без ошибок

### B1.M22

с любых linux/vyos тачек цепляемся по ссх на любую linux тачку

**Аспект + если ВСЕ проверки ОК**

### B1.M23

С любого вин клиента пытаемся цепляться по RDP на ближайший вин-сервер

**Аспект + если ВСЕ проверки ОК**

### B1.M24

На шаре (если Шинда)

```
Get-FileShare
```

Смотрим нужные, dir'ом чекаем внутренности (должны быть папки юзероф)

На клиентах смотрим, что шара приехала

**Аспект + если ВСЕ проверки ОК**
