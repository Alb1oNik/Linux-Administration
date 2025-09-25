# Полное пособие по администрированию Linux серверов

## Оглавление

1. [Введение и основы](#введение-и-основы)
2. [Установка и начальная настройка](#установка-и-начальная-настройка)
3. [Управление пользователями и группами](#управление-пользователями-и-группами)
4. [Файловая система и права доступа](#файловая-система-и-права-доступа)
5. [Управление процессами и службами](#управление-процессами-и-службами)
6. [Сетевое администрирование](#сетевое-администрирование)
7. [Безопасность сервера](#безопасность-сервера)
8. [Мониторинг и логирование](#мониторинг-и-логирование)
9. [Резервное копирование и восстановление](#резервное-копирование-и-восстановление)
10. [Автоматизация и скрипты](#автоматизация-и-скрипты)
11. [Веб-сервисы и базы данных](#веб-сервисы-и-базы-данных)
12. [Виртуализация и контейнеры](#виртуализация-и-контейнеры)
13. [Практические кейсы](#практические-кейсы)

---

## Введение и основы

### Что такое Linux сервер?

Linux сервер — это компьютер, работающий под управлением операционной системы Linux, предназначенный для предоставления услуг другим компьютерам или пользователям в сети. Основные преимущества:

- **Стабильность**: высокая надежность и время работы
- **Безопасность**: встроенные механизмы защиты
- **Производительность**: эффективное использование ресурсов
- **Экономичность**: бесплатность большинства дистрибутивов
- **Гибкость**: широкие возможности настройки

### Популярные серверные дистрибутивы

#### Ubuntu Server
- **Плюсы**: простота установки, большое сообщество, LTS версии
- **Минусы**: может быть избыточным для некоторых задач
- **Применение**: веб-серверы, приложения, облачные решения

#### CentOS/RHEL/Rocky Linux
- **Плюсы**: корпоративная стабильность, долгая поддержка
- **Минусы**: консервативные версии ПО
- **Применение**: корпоративные серверы, критически важные системы

#### Debian
- **Плюсы**: максимальная стабильность, минимализм
- **Минусы**: старые версии пакетов
- **Применение**: серверы высокой надежности, инфраструктурные решения

### Архитектура Linux системы

```
┌─────────────────────────────────────┐
│           Пользователи              │
├─────────────────────────────────────┤
│          Приложения                 │
├─────────────────────────────────────┤
│           Shell                     │
├─────────────────────────────────────┤
│      Системные вызовы               
├─────────────────────────────────────┤
│           Ядро Linux                │
├─────────────────────────────────────┤
│          Аппаратура                 │
└─────────────────────────────────────┘
```

### Основные команды для начинающих

```bash
# Навигация
pwd                 # показать текущую директорию
ls -la             # список файлов с подробностями
cd /path/to/dir    # перейти в директорию
cd ~               # перейти в домашнюю директорию
cd -               # вернуться в предыдущую директорию

# Работа с файлами
cat filename       # показать содержимое файла
less filename      # просмотр файла постранично
head -n 10 file    # первые 10 строк файла
tail -n 10 file    # последние 10 строк файла
tail -f file       # следить за изменениями файла

# Поиск
find /path -name "filename"
grep "pattern" filename
locate filename

# Информация о системе
uname -a           # информация о системе
uptime             # время работы системы
who                # кто подключен к системе
df -h              # использование дисков
free -h            # использование памяти
top                # процессы в реальном времени
```

---

## Установка и начальная настройка

### Планирование установки

#### Системные требования
- **CPU**: минимум 1 ГГц, рекомендуется 2+ ГГц
- **RAM**: минимум 512 МБ, рекомендуется 2+ ГБ
- **Диск**: минимум 10 ГБ, рекомендуется 50+ ГБ
- **Сеть**: Ethernet или Wi-Fi адаптер

#### Схема разбиения диска
```
/boot    - 500 МБ   (загрузочный раздел)
/        - 20 ГБ    (корневая система)
/var     - 10 ГБ    (логи, кэш, данные)
/home    - 10 ГБ    (пользовательские данные)
/tmp     - 2 ГБ     (временные файлы)
swap     - 2x RAM   (виртуальная память)
```

### Процесс установки Ubuntu Server

#### 1. Создание загрузочного носителя
```bash
# Linux
dd if=ubuntu-server.iso of=/dev/sdX bs=4M status=progress

# macOS
dd if=ubuntu-server.iso of=/dev/diskX bs=4m

# Windows - использовать Rufus или Etcher
```

#### 2. Сетевые настройки
```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

#### 3. Применение сетевых настроек
```bash
sudo netplan try
sudo netplan apply
```

### Первоначальная настройка сервера

#### 1. Обновление системы
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
sudo reboot
```

#### 2. Настройка часового пояса
```bash
sudo timedatectl set-timezone Europe/Moscow
timedatectl status
```

#### 3. Настройка hostname
```bash
sudo hostnamectl set-hostname myserver
sudo nano /etc/hosts
# Добавить:
# 127.0.0.1 myserver
```

#### 4. Создание нового пользователя
```bash
sudo adduser admin
sudo usermod -aG sudo admin
su - admin
```

#### 5. Настройка SSH
```bash
sudo nano /etc/ssh/sshd_config

# Рекомендуемые настройки:
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers admin
ClientAliveInterval 300
ClientAliveCountMax 2

sudo systemctl restart ssh
```

---

## Управление пользователями и группами

### Создание и управление пользователями

#### Создание пользователя
```bash
# Интерактивное создание
sudo adduser username

# Создание без интерактива
sudo useradd -m -s /bin/bash username
sudo passwd username

# Создание системного пользователя
sudo useradd -r -s /bin/false service_user
```

#### Модификация пользователя
```bash
# Изменение оболочки
sudo usermod -s /bin/zsh username

# Добавление в группы
sudo usermod -aG sudo,docker,www-data username

# Блокировка пользователя
sudo usermod -L username

# Разблокировка
sudo usermod -U username

# Изменение домашней директории
sudo usermod -d /new/home/dir -m username
```

#### Удаление пользователя
```bash
# Удаление пользователя и домашней папки
sudo deluser --remove-home username

# Или через userdel
sudo userdel -r username
```

### Управление группами

#### Создание и управление группами
```bash
# Создание группы
sudo groupadd developers

# Добавление пользователя в группу
sudo usermod -aG developers username

# Удаление пользователя из группы
sudo deluser username developers

# Удаление группы
sudo groupdel developers

# Просмотр групп пользователя
groups username
id username
```

### Sudo и права доступа

#### Настройка sudo
```bash
sudo visudo

# Примеры настроек:
# Пользователь может выполнять все команды
username ALL=(ALL:ALL) ALL

# Группа без ввода пароля
%admin ALL=(ALL) NOPASSWD: ALL

# Ограниченные права
username ALL=(ALL) /usr/bin/systemctl, /usr/bin/mount

# Алиасы команд
Cmnd_Alias SERVICES = /usr/bin/systemctl, /usr/sbin/service
username ALL=(ALL) SERVICES
```

#### Переключение между пользователями
```bash
su - username          # полное переключение
sudo -u username bash   # выполнение от имени пользователя
sudo -i                # root shell с environment
sudo -s                # root shell без environment
```

---

## Файловая система и права доступа

### Структура файловой системы Linux

```
/                    # Корневая директория
├── bin/            # Основные исполняемые файлы
├── boot/           # Загрузочные файлы
├── dev/            # Файлы устройств
├── etc/            # Конфигурационные файлы
├── home/           # Домашние директории пользователей
├── lib/            # Системные библиотеки
├── media/          # Точки монтирования съемных носителей
├── mnt/            # Временные точки монтирования
├── opt/            # Дополнительное ПО
├── proc/           # Виртуальная файловая система процессов
├── root/           # Домашняя директория root
├── run/            # Временные файлы времени выполнения
├── sbin/           # Системные исполняемые файлы
├── srv/            # Данные сервисов
├── sys/            # Виртуальная файловая система
├── tmp/            # Временные файлы
├── usr/            # Пользовательские программы
└── var/            # Переменные данные
```

### Права доступа к файлам

#### Понимание прав доступа
```bash
ls -l filename
# -rwxr-xr-- 1 user group 1024 Jan 01 12:00 filename
# |  |  |
# |  |  └── права для остальных (r--)
# |  └───── права для группы (r-x)
# └──────── права для владельца (rwx)

# r = 4 (чтение)
# w = 2 (запись)  
# x = 1 (выполнение)
```

#### Управление правами
```bash
# Символьный способ
chmod u+x filename      # добавить выполнение владельцу
chmod g-w filename      # убрать запись у группы
chmod o=r filename      # установить только чтение для остальных
chmod a+r filename      # добавить чтение всем

# Числовой способ
chmod 755 filename      # rwxr-xr-x
chmod 644 filename      # rw-r--r--
chmod 600 filename      # rw-------
chmod 777 filename      # rwxrwxrwx (не рекомендуется)

# Рекурсивно
chmod -R 755 directory/
```

#### Владельцы файлов
```bash
# Изменение владельца
sudo chown user:group filename
sudo chown user filename
sudo chown :group filename

# Рекурсивно
sudo chown -R user:group directory/

# Изменение только группы
sudo chgrp group filename
```

### Специальные права доступа

#### SUID, SGID, Sticky bit
```bash
# SUID (Set User ID) - выполнение от имени владельца
chmod 4755 filename
chmod u+s filename

# SGID (Set Group ID) - выполнение от имени группы
chmod 2755 filename
chmod g+s filename

# Sticky bit - только владелец может удалять файлы
chmod 1755 directory/
chmod +t directory/

# Проверка специальных прав
ls -l /usr/bin/passwd  # -rwsr-xr-x (SUID)
ls -ld /tmp            # drwxrwxrwt (sticky bit)
```

### Управление дисками и разделами

#### Просмотр дисков и разделов
```bash
# Информация о дисках
lsblk
fdisk -l
parted -l

# Использование дискового пространства
df -h
du -sh /path/to/directory
ncdu /path/to/directory  # интерактивный анализ
```

#### Создание разделов
```bash
# Использование fdisk
sudo fdisk /dev/sdb
# n - новый раздел
# p - primary
# w - записать изменения

# Использование parted
sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) quit
```

#### Форматирование разделов
```bash
# ext4
sudo mkfs.ext4 /dev/sdb1

# XFS
sudo mkfs.xfs /dev/sdb1

# NTFS
sudo mkfs.ntfs /dev/sdb1
```

#### Монтирование разделов
```bash
# Временное монтирование
sudo mkdir /mnt/disk
sudo mount /dev/sdb1 /mnt/disk

# Постоянное монтирование через fstab
sudo nano /etc/fstab
# Добавить строку:
# /dev/sdb1 /mnt/disk ext4 defaults 0 2

# Применить настройки fstab
sudo mount -a

# Размонтирование
sudo umount /mnt/disk
```

### LVM (Logical Volume Manager)

#### Создание LVM
```bash
# Создание физического тома
sudo pvcreate /dev/sdb1

# Создание группы томов
sudo vgcreate myvg /dev/sdb1

# Создание логического тома
sudo lvcreate -L 10G -n mylv myvg

# Форматирование и монтирование
sudo mkfs.ext4 /dev/myvg/mylv
sudo mount /dev/myvg/mylv /mnt/logical
```

#### Управление LVM
```bash
# Просмотр информации
pvdisplay
vgdisplay
lvdisplay

# Расширение логического тома
sudo lvextend -L +5G /dev/myvg/mylv
sudo resize2fs /dev/myvg/mylv  # для ext4

# Создание снапшота
sudo lvcreate -L 1G -s -n mylv-snapshot /dev/myvg/mylv
```

---

## Управление процессами и службами

### Просмотр и управление процессами

#### Мониторинг процессов
```bash
# Просмотр запущенных процессов
ps aux              # все процессы
ps -ef              # полная информация
pstree              # древовидная структура

# Интерактивный мониторинг
top                 # классический топ
htop                # улучшенная версия
iotop               # мониторинг I/O
iftop               # мониторинг сети

# Поиск процессов
pgrep nginx         # найти PID процесса nginx
pidof nginx         # альтернативный способ
ps aux | grep nginx # поиск через grep
```

#### Управление процессами
```bash
# Завершение процессов
kill PID            # SIGTERM (мягкое завершение)
kill -9 PID         # SIGKILL (принудительное завершение)
killall nginx       # завершить все процессы nginx
pkill -f "python script.py"  # завершить по части команды

# Сигналы процессам
kill -HUP PID       # перечитать конфигурацию
kill -STOP PID      # приостановить процесс
kill -CONT PID      # возобновить процесс

# Запуск в фоне
command &           # запуск в фоне
nohup command &     # запуск с отключением от терминала
disown              # отключить от текущей сессии

# Управление заданиями
jobs                # активные задания
fg %1               # перевести задание в foreground
bg %1               # перевести в background
```

### systemd - современная система инициализации

#### Основные команды systemctl
```bash
# Управление службами
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx     # автозапуск
sudo systemctl disable nginx    # отключить автозапуск

# Статус служб
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx

# Просмотр всех служб
systemctl list-units --type=service
systemctl list-unit-files --type=service
```

#### Создание собственных служб
```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=myuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# Перезагрузка конфигурации systemd
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
```

#### Логирование с journald
```bash
# Просмотр логов
journalctl                          # все логи
journalctl -u nginx                 # логи службы nginx
journalctl -f                       # следить за логами
journalctl --since "2 hours ago"    # логи за последние 2 часа
journalctl --until "2023-01-01"     # логи до даты

# Фильтрация логов
journalctl -p err                   # только ошибки
journalctl -k                       # kernel логи
journalctl --disk-usage             # использование диска

# Очистка логов
sudo journalctl --vacuum-time=2weeks
sudo journalctl --vacuum-size=100M
```

### Планирование задач

#### Cron - планировщик задач
```bash
# Редактирование crontab
crontab -e          # для текущего пользователя
sudo crontab -e     # для root

# Просмотр crontab
crontab -l
sudo crontab -l

# Формат crontab
# * * * * * command
# | | | | |
# | | | | └── день недели (0-6, воскресенье = 0)
# | | | └──── месяц (1-12)
# | | └────── день месяца (1-31)
# | └──────── час (0-23)
# └────────── минута (0-59)

# Примеры
0 2 * * * /usr/bin/backup.sh           # каждый день в 2:00
*/15 * * * * /usr/bin/check-status.sh  # каждые 15 минут
0 0 * * 0 /usr/bin/weekly-cleanup.sh   # каждое воскресенье в полночь
```

#### Systemd Timers (современная альтернатива cron)
```bash
# Создание таймера
sudo nano /etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Run backup daily
Requires=backup.service

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Создание соответствующего сервиса
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Backup service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Просмотр таймеров
systemctl list-timers
```

#### at - одноразовые задачи
```bash
# Установка
sudo apt install at

# Запуск задачи
at 15:30
at> /usr/bin/backup.sh
at> <Ctrl+D>

# Запуск через время
at now + 1 hour
at 2pm tomorrow

# Просмотр очереди
atq

# Удаление задачи
atrm JOB_NUMBER
```

---

## Сетевое администрирование

### Сетевые интерфейсы и конфигурация

#### Просмотр сетевых интерфейсов
```bash
# Информация об интерфейсах
ip addr show                # показать все интерфейсы
ip link show                # показать состояние интерфейсов
ifconfig                    # старый способ (требует net-tools)

# Статистика интерфейсов
ip -s link show eth0        # статистика для eth0
cat /proc/net/dev           # статистика всех интерфейсов
```

#### Настройка IP адресов
```bash
# Временная настройка
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.1.1

# Удаление IP адреса
sudo ip addr del 192.168.1.100/24 dev eth0
```

#### Netplan (Ubuntu 18.04+)
```yaml
# /etc/netplan/01-network-manager-all.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    eth1:
      dhcp4: yes
```

```bash
sudo netplan try      # тестирование конфигурации
sudo netplan apply    # применение конфигурации
```

#### Традиционная настройка (Debian/старые системы)
```bash
# /etc/network/interfaces
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

# Перезапуск сети
sudo systemctl restart networking
```

### Маршрутизация

#### Просмотр таблицы маршрутизации
```bash
ip route show           # показать все маршруты
route -n                # старый способ
netstat -rn             # альтернативный способ
```

#### Управление маршрутами
```bash
# Добавление маршрута
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route add default via 192.168.1.1

# Удаление маршрута
sudo ip route del 10.0.0.0/8
sudo ip route del default

# Маршрут через конкретный интерфейс
sudo ip route add 172.16.0.0/16 dev eth1
```

### DNS настройка

#### Настройка DNS серверов
```bash
# /etc/resolv.conf (может быть перезаписан)
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com

# Постоянная настройка через systemd-resolved
sudo nano /etc/systemd/resolved.conf
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1 1.0.0.1
```

#### Проверка DNS
```bash
nslookup google.com
dig google.com
host google.com

# Детальная информация
dig +trace google.com
dig @8.8.8.8 google.com
```

### Файрвол и iptables

#### ufw (Uncomplicated Firewall)
```bash
# Включение/выключение
sudo ufw enable
sudo ufw disable

# Основные правила
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow from 192.168.1.0/24

# Блокировка
sudo ufw deny 23/tcp
sudo ufw deny from 10.0.0.5

# Просмотр правил
sudo ufw status verbose
sudo ufw status numbered

# Удаление правил
sudo ufw delete 2
sudo ufw delete allow ssh
```

#### iptables (расширенная настройка)
```bash
# Просмотр правил
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v

# Разрешение входящих подключений
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Разрешение с определенного IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Блокировка
sudo iptables -A INPUT -s 10.0.0.5 -j DROP

# NAT (Network Address Translation)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# Сохранение правил
sudo iptables-save > /etc/iptables/rules.v4
```

### Сетевые службы

#### SSH сервер
```bash
# Установка
sudo apt install openssh-server

# Конфигурация /etc/ssh/sshd_config
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers user1 user2

# Перезапуск
sudo systemctl restart ssh
```

#### SSH ключи
```bash
# Генерация ключей на клиенте
ssh-keygen -t rsa -b 4096 -C "user@example.com"

# Копирование ключа на сервер
ssh-copy-id user@server
# или вручную
cat ~/.ssh/id_rsa.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Подключение
ssh user@server -p 2222
```

### Мониторинг сети

#### Сетевые утилиты
```bash
# Тестирование связности
ping google.com
ping6 google.com

# Трассировка маршрута
traceroute google.com
mtr google.com           # интерактивная трассировка

# Сканирование портов
nmap localhost
nmap -p 1-1000 192.168.1.1
nmap -sP 192.168.1.0/24  # поиск активных хостов

# Мониторинг соединений
netstat -tulpn           # активные порты
ss -tulpn                # современная альтернатива
lsof -i :80              # что использует порт 80
```

#### tcpdump - анализ трафика
```bash
# Основное использование
sudo tcpdump -i eth0
sudo tcpdump -i any port 80

# Фильтры
sudo tcpdump host 192.168.1.100
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 192.168.1.100

# Сохранение в файл
sudo tcpdump -i eth0 -w capture.pcap
tcpdump -r capture.pcap
```

---

## Безопасность сервера

### Базовая безопасность

#### Обновления системы
```bash
# Автоматические обновления (Ubuntu)
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Конфигурация автообновлений
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

#### Настройка SSH безопасности
```bash
# /etc/ssh/sshd_config
Protocol 2
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
X11Forwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers username
DenyUsers baduser
```

#### Fail2Ban - защита от брутфорса
```bash
# Установка
sudo apt install fail2ban

# Конфигурация
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log

# Управление fail2ban
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo fail2ban-client unban IP_ADDRESS
```

### Аудит и мониторинг безопасности

#### Логирование действий пользователей
```bash
# Установка auditd
sudo apt install auditd audispd-plugins

# Конфигурация /etc/audit/rules.d/audit.rules
# Мониторинг файлов
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/sudoers -p wa -k sudoers_changes

# Мониторинг команд
-a always,exit -F arch=b64 -S execve -k commands
-a always,exit -F arch=b32 -S execve -k commands

# Перезапуск auditd
sudo systemctl restart auditd

# Просмотр логов
sudo ausearch -k passwd_changes
sudo aureport --summary
```

#### Сканирование на уязвимости
```bash
# Установка и использование Lynis
sudo apt install lynis
sudo lynis audit system

# Chkrootkit - поиск руткитов
sudo apt install chkrootkit
sudo chkrootkit

# RKHunter - еще один сканер руткитов
sudo apt install rkhunter
sudo rkhunter --update
sudo rkhunter --check
```

#### AIDE - система обнаружения вторжений
```bash
# Установка
sudo apt install aide

# Инициализация базы данных
sudo aide --init
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Проверка целостности
sudo aide --check

# Автоматическая проверка через cron
echo "0 2 * * * root /usr/bin/aide --check" | sudo tee -a /etc/crontab
```

### Шифрование и сертификаты

#### GPG - шифрование файлов
```bash
# Генерация ключей
gpg --gen-key

# Экспорт публичного ключа
gpg --export -a "Your Name" > public.key

# Шифрование файла
gpg --encrypt --recipient "recipient@example.com" file.txt

# Расшифровка
gpg --decrypt file.txt.gpg

# Подпись файла
gpg --sign file.txt
gpg --verify file.txt.gpg
```

#### SSL/TLS сертификаты с Let's Encrypt
```bash
# Установка Certbot
sudo apt install certbot python3-certbot-nginx

# Получение сертификата для nginx
sudo certbot --nginx -d example.com -d www.example.com

# Автоматическое обновление
sudo crontab -e
# Добавить:
0 12 * * * /usr/bin/certbot renew --quiet

# Тестирование обновления
sudo certbot renew --dry-run
```

### Контроль доступа

#### SELinux (в системах RHEL/CentOS)
```bash
# Проверка статуса
getenforce
sestatus

# Режимы SELinux
sudo setenforce 0    # Permissive mode
sudo setenforce 1    # Enforcing mode

# Постоянное изменение в /etc/selinux/config
SELINUX=enforcing

# Управление контекстами
ls -Z /var/www/html/
chcon -t httpd_exec_t /var/www/html/script.cgi
restorecon -Rv /var/www/html/

# Политики SELinux
getsebool -a | grep httpd
setsebool -P httpd_can_network_connect on
```

#### AppArmor (Ubuntu/Debian)
```bash
# Статус AppArmor
sudo apparmor_status

# Профили
ls /etc/apparmor.d/

# Режимы профилей
sudo aa-enforce /etc/apparmor.d/usr.bin.firefox
sudo aa-complain /etc/apparmor.d/usr.bin.firefox
sudo aa-disable /etc/apparmor.d/usr.bin.firefox

# Создание профиля
sudo aa-genprof /usr/bin/myapp
```

---

## Мониторинг и логирование

### Системный мониторинг

#### Мониторинг ресурсов
```bash
# CPU и память
top
htop
iotop
vmstat 1
mpstat 1

# Дисковая активность
iostat -x 1
iotop
df -h
du -sh /*

# Сетевая активность
iftop
nethogs
ss -tulpn
netstat -i
```

#### Расширенный мониторинг с помощью инструментов

##### Установка и настройка Zabbix Agent
```bash
# Установка агента
sudo apt install zabbix-agent

# Конфигурация /etc/zabbix/zabbix_agentd.conf
Server=192.168.1.100
ServerActive=192.168.1.100
Hostname=webserver01

sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```

##### Prometheus Node Exporter
```bash
# Установка
sudo useradd --no-create-home --shell /bin/false node_exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.1.linux-amd64.tar.gz
sudo cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

# Создание systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

### Централизованное логирование

#### rsyslog - конфигурация
```bash
# /etc/rsyslog.conf
# Локальные логи
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
authpriv.*                                  /var/log/secure
mail.*                                      /var/log/maillog
cron.*                                      /var/log/cron

# Отправка логов на удаленный сервер
*.* @@logserver.example.com:514

# Прием логов от других серверов
$ModLoad imudp
$UDPServerRun 514
$UDPServerAddress 0.0.0.0

sudo systemctl restart rsyslog
```

#### Logrotate - ротация логов
```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 myapp myapp
    postrotate
        systemctl reload myapp
    endscript
}

# Тестирование конфигурации
sudo logrotate -d /etc/logrotate.d/myapp

# Принудительная ротация
sudo logrotate -f /etc/logrotate.d/myapp
```

#### ELK Stack (Elasticsearch, Logstash, Kibana)

##### Установка Elasticsearch
```bash
# Добавление репозитория
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
sudo apt install elasticsearch

# Конфигурация /etc/elasticsearch/elasticsearch.yml
cluster.name: my-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node

sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

##### Установка Logstash
```bash
sudo apt install logstash

# Конфигурация /etc/logstash/conf.d/syslog.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fileset][module] == "system" {
    if [fileset][name] == "auth" {
      grok {
        match => { "message" => ["%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{IPORHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: %{DATA:[system][auth][ssh][event]} %{DATA:[system][auth][ssh][method]} for (invalid user )?%{DATA:[system][auth][user]} from %{IPORHOST:[system][auth][ssh][ip]} port %{INT:[system][auth][ssh][port]} ssh2(: %{GREEDYDATA:[system][auth][ssh][signature]})?"] }
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}

sudo systemctl start logstash
sudo systemctl enable logstash
```

### Алертинг и уведомления

#### Мониторинг с помощью скриптов
```bash
#!/bin/bash
# /usr/local/bin/check_disk_space.sh

THRESHOLD=90
PARTITION="/"

USAGE=$(df -h "$PARTITION" | awk 'NR==2 {print $(NF-1)}' | sed 's/%//')

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "Disk usage on $PARTITION is ${USAGE}% - exceeds threshold of ${THRESHOLD}%"
    # Отправка email
    echo "Disk space warning on $(hostname)" | mail -s "Disk Space Alert" admin@example.com
    
    # Отправка в Slack
    curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"Disk usage alert on '$(hostname)': '${USAGE}'%"}' \
        YOUR_SLACK_WEBHOOK_URL
fi
```

#### Настройка email уведомлений
```bash
# Установка и настройка Postfix
sudo apt install postfix mailutils

# Конфигурация /etc/postfix/main.cf
myhostname = server.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = loopback-only
mydestination = $myhostname, localhost.$mydomain, localhost

# Перезапуск
sudo systemctl restart postfix

# Тестирование
echo "Test message" | mail -s "Test Subject" user@example.com
```

---

## Резервное копирование и восстановление

### Стратегии резервного копирования

#### Типы резервных копий
- **Полная копия (Full Backup)**: полная копия всех данных
- **Инкрементальная (Incremental)**: только изменения с последней копии
- **Дифференциальная (Differential)**: изменения с последней полной копии

#### Правило 3-2-1
- **3** копии данных (оригинал + 2 копии)
- **2** разных типа носителей
- **1** копия в другом географическом месте

### Инструменты резервного копирования

#### rsync - синхронизация файлов
```bash
# Локальная синхронизация
rsync -av /source/ /destination/

# Удаленная синхронизация
rsync -av /local/path/ user@remote:/remote/path/

# Исключение файлов
rsync -av --exclude='*.tmp' --exclude='cache/' /source/ /dest/

# Удаление файлов в назначении
rsync -av --delete /source/ /destination/

# Резервная копия с сохранением прав и атрибутов
rsync -avH --numeric-ids /source/ /backup/

# Ограничение скорости
rsync -av --bwlimit=1000 /source/ /destination/
```

#### tar - архивирование
```bash
# Создание архива
tar -czf backup_$(date +%Y%m%d).tar.gz /path/to/backup/

# Создание с исключениями
tar -czf backup.tar.gz --exclude='*.log' --exclude='tmp/*' /path/to/backup/

# Извлечение архива
tar -xzf backup.tar.gz

# Просмотр содержимого
tar -tzf backup.tar.gz

# Инкрементальное архивирование
tar -czf full_backup.tar.gz -g snapshot.snar /path/to/backup/
tar -czf incremental_backup.tar.gz -g snapshot.snar /path/to/backup/
```

#### dd - побитовое копирование
```bash
# Клонирование диска
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress

# Создание образа диска
sudo dd if=/dev/sda of=/backup/disk_image.img bs=4M

# Восстановление из образа
sudo dd if=/backup/disk_image.img of=/dev/sda bs=4M

# Создание образа раздела
sudo dd if=/dev/sda1 of=/backup/partition.img bs=4M
```

### Автоматизированные решения

#### Скрипт резервного копирования
```bash
#!/bin/bash
# /usr/local/bin/backup.sh

# Переменные
BACKUP_DIR="/backup"
SOURCE_DIRS="/etc /var/www /home"
DATE=$(date +%Y%m%d_%H%M%S)
HOSTNAME=$(hostname)
RETENTION_DAYS=30

# Создание директории
mkdir -p "$BACKUP_DIR/$HOSTNAME"

# Создание архива
tar -czf "$BACKUP_DIR/$HOSTNAME/backup_$DATE.tar.gz" $SOURCE_DIRS

# Удаление старых копий
find "$BACKUP_DIR/$HOSTNAME" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

# Проверка успешности
if [ $? -eq 0 ]; then
    echo "Backup completed successfully: backup_$DATE.tar.gz"
    # Отправка уведомления об успехе
    echo "Backup completed on $HOSTNAME" | mail -s "Backup Success" admin@example.com
else
    echo "Backup failed"
    # Отправка уведомления об ошибке
    echo "Backup failed on $HOSTNAME" | mail -s "Backup Failed" admin@example.com
fi

# Логирование
echo "$(date): Backup operation completed" >> /var/log/backup.log
```

#### Duplicity - зашифрованные резервные копии
```bash
# Установка
sudo apt install duplicity

# Полная резервная копия
duplicity /home/user file:///backup/duplicity/

# Инкрементальная копия
duplicity incr /home/user file:///backup/duplicity/

# Восстановление
duplicity restore file:///backup/duplicity/ /restore/path/

# Восстановление определенного файла
duplicity restore --file-to-restore path/to/file file:///backup/duplicity/ /restore/path/

# Удаление старых копий
duplicity remove-older-than 1M file:///backup/duplicity/
```

#### Bacula - корпоративное решение
```bash
# Установка сервера Bacula
sudo apt install bacula-server bacula-client

# Основные компоненты:
# - Bacula Director (управление)
# - Storage Daemon (хранение)
# - File Daemon (клиент)

# Конфигурация директора /etc/bacula/bacula-dir.conf
Director {
  Name = backup-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "director_password"
  Messages = Daemon
}

Job {
  Name = "backup-job"
  Type = Backup
  Level = Incremental
  Client = backup-fd
  FileSet = "Full Set"
  Schedule = "WeeklyCycle"
  Storage = File1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}
```

### Восстановление системы

#### Восстановление из резервной копии
```bash
# Восстановление из tar архива
cd /
sudo tar -xzf /backup/system_backup.tar.gz

# Восстановление определенных файлов
sudo tar -xzf /backup/system_backup.tar.gz etc/passwd etc/shadow

# Восстановление с rsync
sudo rsync -av /backup/system/ /

# Восстановление прав доступа
sudo chown -R root:root /etc/
sudo chmod 644 /etc/passwd
sudo chmod 600 /etc/shadow
```

#### Восстановление загрузчика GRUB
```bash
# Загрузка с Live CD/USB
sudo mount /dev/sda1 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys

# Chroot в систему
sudo chroot /mnt

# Переустановка GRUB
grub-install /dev/sda
update-grub

# Выход из chroot
exit
sudo umount /mnt/dev /mnt/proc /mnt/sys /mnt
```

---

## Автоматизация и скрипты

### Bash скриптинг для администраторов

#### Основы bash скриптов
```bash
#!/bin/bash
# Шебанг - указывает интерпретатор

# Переменные
NAME="Server Admin"
DATE=$(date +%Y-%m-%d)
COUNT=10

# Массивы
SERVERS=("web1" "web2" "db1")
PORTS=(80 443 22 3306)

# Условия
if [ -f "/etc/passwd" ]; then
    echo "Password file exists"
elif [ -f "/etc/shadow" ]; then
    echo "Shadow file exists"
else
    echo "No password files found"
fi

# Циклы
for server in "${SERVERS[@]}"; do
    echo "Checking $server"
    ping -c 1 "$server" >/dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "$server is online"
    else
        echo "$server is offline"
    fi
done

# While цикл
counter=1
while [ $counter -le 5 ]; do
    echo "Iteration: $counter"
    ((counter++))
done

# Функции
check_service() {
    local service_name=$1
    if systemctl is-active --quiet "$service_name"; then
        echo "$service_name is running"
        return 0
    else
        echo "$service_name is not running"
        return 1
    fi
}

# Использование функции
check_service "nginx"
```

#### Практические скрипты для администрирования

##### Скрипт мониторинга системы
```bash
#!/bin/bash
# /usr/local/bin/system_check.sh

LOG_FILE="/var/log/system_check.log"
EMAIL="admin@example.com"

log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

check_disk_space() {
    local threshold=90
    local usage=$(df -h / | awk 'NR==2 {print $(NF-1)}' | sed 's/%//')
    
    if [ "$usage" -gt "$threshold" ]; then
        log_message "WARNING: Disk usage is ${usage}%"
        echo "Disk usage warning: ${usage}%" | mail -s "Disk Alert - $(hostname)" "$EMAIL"
    else
        log_message "INFO: Disk usage is ${usage}%"
    fi
}

check_memory() {
    local mem_usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    local threshold=85
    
    if [ "$mem_usage" -gt "$threshold" ]; then
        log_message "WARNING: Memory usage is ${mem_usage}%"
        echo "Memory usage warning: ${mem_usage}%" | mail -s "Memory Alert - $(hostname)" "$EMAIL"
    else
        log_message "INFO: Memory usage is ${mem_usage}%"
    fi
}

check_load() {
    local load=$(uptime | awk '{print $10}' | sed 's/,//')
    local cpu_count=$(nproc)
    local threshold=$(echo "$cpu_count * 2" | bc)
    
    if (( $(echo "$load > $threshold" | bc -l) )); then
        log_message "WARNING: Load average is $load"
        echo "Load average warning: $load" | mail -s "Load Alert - $(hostname)" "$EMAIL"
    else
        log_message "INFO: Load average is $load"
    fi
}

check_services() {
    local services=("nginx" "mysql" "ssh")
    
    for service in "${services[@]}"; do
        if ! systemctl is-active --quiet "$service"; then
            log_message "ERROR: Service $service is not running"
            echo "Service $service is down" | mail -s "Service Alert - $(hostname)" "$EMAIL"
            # Попытка перезапуска
            systemctl start "$service"
            if systemctl is-active --quiet "$service"; then
                log_message "INFO: Service $service restarted successfully"
            else
                log_message "ERROR: Failed to restart service $service"
            fi
        else
            log_message "INFO: Service $service is running"
        fi
    done
}

# Выполнение проверок
log_message "Starting system checks"
check_disk_space
check_memory
check_load
check_services
log_message "System checks completed"
```

##### Скрипт управления пользователями
```bash
#!/bin/bash
# /usr/local/bin/user_management.sh

LOGFILE="/var/log/user_management.log"

log_action() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOGFILE"
}

create_user() {
    local username=$1
    local fullname=$2
    local groups=$3
    
    if id "$username" &>/dev/null; then
        echo "User $username already exists"
        return 1
    fi
    
    useradd -m -c "$fullname" -s /bin/bash "$username"
    
    if [ -n "$groups" ]; then
        usermod -aG "$groups" "$username"
    fi
    
    # Генерация временного пароля
    temp_password=$(openssl rand -base64 12)
    echo "$username:$temp_password" | chpasswd
    
    # Принудительная смена пароля при первом входе
    chage -d 0 "$username"
    
    log_action "User $username created with groups: $groups"
    echo "User $username created. Temporary password: $temp_password"
    echo "User must change password on first login."
}

disable_user() {
    local username=$1
    
    if ! id "$username" &>/dev/null; then
        echo "User $username does not exist"
        return 1
    fi
    
    usermod -L "$username"
    usermod -s /bin/false "$username"
    
    log_action "User $username disabled"
    echo "User $username has been disabled"
}

list_inactive_users() {
    local days=${1:-30}
    
    echo "Users inactive for more than $days days:"
    lastlog -b "$days" | grep -v "Never logged in" | awk 'NR>1 {print $1}'
}

# Меню
case "$1" in
    create)
        create_user "$2" "$3" "$4"
        ;;
    disable)
        disable_user "$2"
        ;;
    inactive)
        list_inactive_users "$2"
        ;;
    *)
        echo "Usage: $0 {create|disable|inactive}"
        echo "  create username 'Full Name' 'groups'"
        echo "  disable username"
        echo "  inactive [days]"
        exit 1
        ;;
esac
```

### Ansible - автоматизация конфигурации

#### Установка и настройка Ansible
```bash
# Установка
sudo apt install ansible

# Или через pip
pip3 install ansible

# Структура проекта
mkdir ansible-project
cd ansible-project
mkdir inventory playbooks roles group_vars host_vars
```

#### Inventory файл
```ini
# inventory/hosts
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

#### Простой playbook
```yaml
# playbooks/webserver.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  
  vars:
    nginx_port: 80
    
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Start and enable nginx
      systemd:
        name: nginx
        state: started
        enabled: yes
    
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        backup: yes
      notify: restart nginx
    
    - name: Open firewall for nginx
      ufw:
        rule: allow
        port: "{{ nginx_port }}"
        proto: tcp
  
  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
```

#### Запуск playbook
```bash
# Проверка синтаксиса
ansible-playbook --syntax-check playbooks/webserver.yml

# Сухой прогон
ansible-playbook --check playbooks/webserver.yml -i inventory/hosts

# Выполнение
ansible-playbook playbooks/webserver.yml -i inventory/hosts

# Выполнение с дополнительными переменными
ansible-playbook playbooks/webserver.yml -i inventory/hosts --extra-vars "nginx_port=8080"
```

### Terraform - инфраструктура как код

#### Основы Terraform
```hcl
# main.tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
  key_name      = "my-key"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
    #!/bin/bash
    apt update
    apt install -y nginx
    systemctl start nginx
    systemctl enable nginx
  EOF
  
  tags = {
    Name = "Web Server"
    Environment = "Production"
  }
}

resource "aws_security_group" "web" {
  name_prefix = "web-"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

output "instance_ip" {
  value = aws_instance.web_server.public_ip
}
```

---

## Веб-сервисы и базы данных

### Веб-серверы

#### Apache HTTP Server
```bash
# Установка
sudo apt install apache2

# Основные команды
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2

# Конфигурационные файлы
/etc/apache2/apache2.conf          # Основная конфигурация
/etc/apache2/sites-available/      # Доступные сайты
/etc/apache2/sites-enabled/        # Включенные сайты
/etc/apache2/mods-available/       # Доступные модули
/etc/apache2/mods-enabled/         # Включенные модули
```

Конфигурация виртуального хоста:
```apache
# /etc/apache2/sites-available/example.com.conf
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com
    
    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined
    
    <Directory /var/www/example.com>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

```bash
# Включение сайта
sudo a2ensite example.com.conf
sudo systemctl reload apache2

# Включение