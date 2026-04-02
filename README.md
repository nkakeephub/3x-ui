# Установка и настройка панели 3X-UI в режиме каскада (XRAY VLESS)

> [!IMPORTANT]
> Этот проект предназначен только для **личного использования**.  
> Данный материал создан *исключительно в образовательных целях*.  
> Автор не несёт ответственности за возможные последствия использования.  
> Применяйте только в рамках законного тестирования и легальных задач.

## Обновление пакетов:
    apt update && apt upgrade -y

## Установка утилиты командной строки:
    apt install wget curl -y

## Установка интерфейса для управления правилами брандмауэра:
    apt install ufw

## Включение фаервола:
    ufw enable

## Добавление портов в фаервол:
    ufw allow 22
    ufw allow 80/tcp

## Команда покажет состояние брандмауэра и открытых портах:
    ufw status verbose

## Создадим файл настроек:
### 1. Открой файл:
    sudo nano /etc/sysctl.conf
### 2. Удали оттуда всё (можешь просто зажать Ctrl+K, пока файл не опустеет) и вставь этот блок:
\# Тюнинг скорости (BBR + FQ + FastOpen)

net.core.default_qdisc = fq

net.ipv4.tcp_congestion_control = bbr

net.ipv4.tcp_fastopen = 3

\# Оптимизация буферов

net.core.rmem_max = 67108864

net.core.wmem_max = 67108864

net.ipv4.tcp_rmem = 4096 87380 67108864

net.ipv4.tcp_wmem = 4096 65536 67108864

\# Лимиты очередей

net.core.somaxconn = 4096

net.core.netdev_max_backlog = 5000

net.ipv4.tcp_max_syn_backlog = 4096

\# Безопасность

net.ipv4.conf.all.rp_filter = 1

net.ipv4.conf.default.rp_filter = 1

net.ipv4.icmp_echo_ignore_broadcasts = 1

net.ipv4.tcp_syncookies = 1

\# DPI-friendly

net.ipv4.tcp_mtu_probing = 1

\# Стабильность соединений

net.ipv4.tcp_keepalive_time = 600

net.ipv4.tcp_keepalive_intvl = 60

net.ipv4.tcp_keepalive_probes = 3

\# TIME_WAIT и FIN

net.ipv4.tcp_fin_timeout = 30

\# Отключение ipv6

net.ipv6.conf.all.disable_ipv6 = 1

net.ipv6.conf.default.disable_ipv6 = 1

net.ipv6.conf.lo.disable_ipv6 = 1

### Сохранить и применить.

### После сохранения выполнить:
    sysctl -p

## Шаг 3: Настройка лимитов Xray

Настройки в sysctl — это «ширина дороги». Но системе нужно еще разрешить Xray открывать много «дверей» (файлов).
### Лимиты файловых дескрипторов, добавьте в 
    nano /etc/security/limits.conf

* soft nofile 1000000

* hard nofile 1000000

### После всех изменений: 
    sudo reboot

## Создадим бэкап настроек:
Мы сделаем копию текущего конфига и сразу создадим файл с дефолтными настройками, чтобы можно было откатиться одной командой.

### 1. Сделать копию:
    sudo cp /etc/sysctl.conf /etc/sysctl.conf.bak
### 2. Сохранить текущие значения «в памяти» (на случай, если захочешь проверить, как было):
    sysctl -a > ~/sysctl_full_backup.txt
### 3: Как откатиться (если что-то пойдет не так):
    sudo mv /etc/sysctl.conf.bak /etc/sysctl.conf && sudo sysctl -p

## Установка панели 3x-ui
    bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)

## Теперь нужно добавить порт в фаервол, который сгенерировала панель, для доступа к панели 3x-ui:
    ufw allow (твой порт)/tcp

## Добавление WARP (для сервера за рубежом)
Настройка Xray>Исходящие подключения>WARP (поставить 2-м в списке) (для заграничного сервера)
Настройка Xray>Маршрутизация>Создать правило
Domain? domain:youtube.com
Outbound Tag? warp

###Подключения>Создать подключение (inbounds)

###Добавить порт в фаервол для доступа к inbounds
ufw allow 443/tcp


Сервер МСК открыть порт 8443
Сервер Бугор открыть порт 443
