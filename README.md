# amneziawg-client
amneziawg-go для Entware

Данный репазитарий является архивом файлов форума keenetic.ru

https://forum.keenetic.ru/topic/18794-amneziawg-go-%D0%B4%D0%BB%D1%8F-entware/

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

Как это запустить если у вас роутер < 4.2:
1. Скачайте и установите для своей архитектуры 2 установочных пакета
2. Запустите `amneziawg-go awg0`  # awg0 - имя будущего интерфейса
3. Задайте адрес интерфейса `ip address add dev awg0 10.7.0.15/24` # 10.7.0.15/24 - ip адрес из полученной конфигурации
4. С помощью конфигурационного файла задайте остальные параметры `awg setconf awg0 ./test.conf`
   Внимание файл конфигурации немного отличается от обычно присылаемого, сохраните копию и отредактируйте.
   Удалите строки содержащие `Address=` и `DNS=` из секции `[Interface]`
6. Далее поднимайте интерфейс `ip link set up dev awg0`

Можете проверить : ` curl --interface awg0 http://myip.wtf/json`   



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
Файл с таким содержимым помещаем в `/opt/etc/init.d`

У меня строка `ip link add dev awg0 type wireguard` вызывает ошибку, но она не нужна для работы.
Далее в КВАСе задаем команду `kvas vpn manual` и вводим имя интерфейса, `awg0` как из данного примера.
