# amneziawg-client
## amneziawg-go для Entware

_Данный репазитарий является архивом файлов из найденой ветки форума keenetic.ru
[amneziawg-go для Entware](https://forum.keenetic.ru/topic/18794-amneziawg-go-%D0%B4%D0%BB%D1%8F-entware/)_


---
Собрал для нескольких архитектур по просьбе трудящихся amneziawg-go  
Пакет(ы) потребуются тем, кто не хочет ставить бету 4.2 и тем, у кого не кинетик. На некоторых роутерах по непонятной причине при запуске паника - https://github.com/Entware/Entware/issues/1078

Проверил на mipsel кинетике - работает.

Вот лог:
```c
root@Keenetic_Ultra:/opt/tmp$ cat test.conf
[Interface]
PrivateKey = .....
ListenPort = 51820
Jc = 1
Jmin = 1
Jmax = 3
S1 = 0
S2 = 0
H1 = 1
H2 = 2
H3 = 3
H4 = 4

[Peer]
PublicKey = ....
PresharedKey = ....
AllowedIPs =  0.0.0.0/0
Endpoint = 23.94.xxxxx:51820
PersistentKeepalive = 25


root@Keenetic_Ultra:/opt/tmp$ amneziawg-go wg0
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│       Running amneziawg-go is not required because this      │
│       kernel has first class support for AmneziaWG. For      │
│       information on installing the kernel module,           │
│       please visit:                                          │
| https://github.com/amnezia-vpn/amneziawg-linux-kernel-module │
│                                                              │
└──────────────────────────────────────────────────────────────┘
root@Keenetic_Ultra:/opt/tmp$ ip address add dev wg0 10.7.0.15/24
root@Keenetic_Ultra:/opt/tmp$ awg setconf wg0 ./test.conf
root@Keenetic_Ultra:/opt/tmp$ ip link set up dev wg0
root@Keenetic_Ultra:/opt/tmp$ /opt/sbin/iptables -A INPUT -i wg0 -j ACCEPT
root@Keenetic_Ultra:/opt/tmp$ /opt/sbin/iptables -A FORWARD -i wg0 -j ACCEPT
root@Keenetic_Ultra:/opt/tmp$  curl --interface wg0 http://myip.wtf/json
{
    "YourFuckingIPAddress": "23.94.xxxxx",
    "YourFuckingLocation": "Amsterdam, NH, The Netherlands",
    "YourFuckingHostname": "23-94-xxxxxxx-host.colocrossing.com",
    "YourFuckingISP": "HostPapa",
    "YourFuckingTorExit": false,
    "YourFuckingCity": "Amsterdam",
    "YourFuckingCountry": "The Netherlands",
    "YourFuckingCountryCode": "NL"
}
```
Ну и собственно пакеты и Makefile тут - http://zyxnerd.zyxmon.org/files/amnezia/

---
Как это запустить если у вас роутер < 4.2:
1. Скачайте и установите для своей архитектуры 2 установочных пакета
2. Запустите `amneziawg-go awg0`  # awg0 - имя будущего интерфейса
3. Задайте адрес интерфейса `ip address add dev awg0 10.7.0.15/24` # 10.7.0.15/24 - ip адрес из полученной конфигурации
4. С помощью конфигурационного файла задайте остальные параметры `awg setconf awg0 ./test.conf`
   Внимание файл конфигурации немного отличается от обычно присылаемого, сохраните копию и отредактируйте.
   Удалите строки содержащие `Address=` и `DNS=` из секции `[Interface]`
6. Поднимайте интерфейс `ip link set up dev awg0`
7. Проверьте : ` curl --interface awg0 http://myip.wtf/json`   

Архитектуру Вы уже знаете когда выбирали файл для [установки Entware](https://help.keenetic.com/hc/ru/articles/360021214160-%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%BE%D0%B2-%D1%80%D0%B5%D0%BF%D0%BE%D0%B7%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D1%8F-Entware-%D0%BD%D0%B0-USB-%D0%BD%D0%B0%D0%BA%D0%BE%D0%BF%D0%B8%D1%82%D0%B5%D0%BB%D1%8C) на флеш карту, если вдруг забыли то самый простой способ прочитать файл `/opt/etc/entware_release`.


Не проверял, но вот такой запуск должен работать:
```
#!/bin/sh

ENABLED=yes
PROCS=amneziawg-go
ARGS="awg0"
PREARGS=""
DESC=$PROCS
PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

pre_cmd() {
  if [ ! -d "/sys/class/net/awg0" ]; then
    ip link add dev awg0 type wireguard
    ip address add dev awg0 10.8.1.4/32
    awg setconf awg0 /opt/etc/awg.conf
    ip link set up dev awg0
  fi
}

PRECMD="pre_cmd"

. /opt/etc/init.d/rc.func
```
Файл с таким содержимым помещаем в `/opt/etc/init.d` имя файла необходимо выбрать такое что бы скрипт запускался ранее КВАС

У меня строка `ip link add dev awg0 type wireguard` вызывает ошибку, но она не нужна для работы.  
Процесс amneziawg-go очень живучий не требует перезапуска, при пропадении связи с сервером даже на длительное время затем самостоятельно востанавливается.  
Далее в КВАСе задаем команду `kvas vpn manual` и вводим имя интерфейса, `awg0` как из данного примера.
