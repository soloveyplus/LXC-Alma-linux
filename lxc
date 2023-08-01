Установка и настройка LXC на Alma Linux. Настройка сети LXC. 

1. Так как работает на мостах:
dnf install epel-release
dnf install bridge-utils

Теперь командой brctl addbr  **
можем добавлять мост

2. Создаем файл будущего моста для серых IP

mcedit /etc/sysconfig/network-scripts/ifcfg-lxcbr0

с содержимым:

DEVICE=lxcbr0
BOOTPROTO=static
IPADDR=10.1.1.1   ### IP моста и серой подсети
NETMASK=255.255.255.0
ONBOOT=yes
TYPE=Bridge

3. Создаем сам мост:

brctl addbr lxcbr0

Мост создан. Перезапускаем  наш сервер. Проверка появился или нет ip a.

4. Для поддержки сети контейнеров присвоения адресов обязательно(для серых) установим:

dnf install dnsmasq

Изменим в /etc/sysconfig/lxc :

с false на true

USE_LXC_BRIDGE="true"

5. Обязательно включить форвардинг:
Для этого в файл /etc/sysctl.conf добавляем строку в самый конец:

net.ipv4.ip_forward = 1

потом:

sysctl -p

6. Установим сам LXC:

dnf install lxc lxc-templates 

Проверим готовность системы к работе lxc:

lxc-checkconfig

Без ругатни.

7. Запускаем:

# systemctl start lxc
# systemctl enable lxc
# systemctl status lxc

8. Запускаем сеть LXC:

systemctl enable --now lxc-net.service

Без ругатни.

9. Пробуем создать контейнер из шаблона:

lxc-create --template download -n mycontainer --  --keyserver hkp://keyserver.ubuntu.com

где,
--keyserver  - нужный ключ скачивания шаблона. Без него никак.

10. Запускаем его:

 lxc-start mycontainer

11. Смотрим что получилось:

lxc-ls -f

IP из серых должен быть присвоен....

11. Как теперь к нему можно обратиться? Ведь он серый.

Даем правила:

Правило проброса портов с хоста в контейнер:
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 20000 -j DNAT --to 192.168.122.47:22
где:
eth0 - интерфейс, смотрящий в инет(хост)
--dport 20000 - номер порта по которому будем стучаться на хост
192.168.122.47:22 - Ip адрес контейнера и номер порта

Но. Перед этим правилом я добавил:

iptables -I FORWARD -j ACCEPT

Потом все заработало!

12. Теперь присвоим контейнеру реальный IP нашей подсети. Для этого:

Создаем конфиг для нового бриджа:
mcedit /etc/sysconfig/network-scripts/ifcfg-virbr0

с содержимым:

DEVICE=virbr0
BOOTPROTO=static
IPADDR=192.168.1.24  ### Реальный ip нашего сервера. статика
NETMASK=255.255.255.0 ###
GATEWAY=192.168.1.1  ### 
DNS1=192.168.1.1  ###
ONBOOT=yes
TYPE=Bridge

И приводим конфиг основного сетевого интерфейса к такому виду:

mcedit /etc/sysconfig/network-scripts/ifcfg-enp1s0

с содержимым:

TYPE=Ethernet
BOOTPROTO=none
DEVICE=enp1s0
ONBOOT=yes
BRIDGE=virbr0

Видим, что теперь все будет ходить через мост virbr0, а наш основной до этого сетевой интерфейс enp1s0 подменяется.

13. Перезапускаем сервер

Должен появится мост с ip virbr0

Короче два моста - virbr0 и lxcbr0

14. Редактируем конфиг нашего контейнера для перевода его на физ. адрес сети:

/var/lib/lxc/название/config

# Network configuration
lxc.net.0.type = veth
lxc.net.0.link = virbr0  ### Наш мост 
lxc.net.0.flags = up
lxc.net.0.hwaddr = 00:16:3e:22:87:81  #### mac можно сгенерить
lxc.net.0.name = eth0  ###  название интерфейса который будет внутри контейнера
lxc.net.0.ipv4.address = 192.168.1.222 #### реальный свободный ip
lxc.net.0.ipv4.gateway = 192.168.1.1   ####  реальный шлюз наше сети

Запускаем. Должно все работать.

Что примечательно, правила в данном случае не создаются. Контейнер работает напрямую.

Меняя адресацию контенеров серый - реальный можно выбрать оптимальный режим изоляции.
------------------------------------------------------












