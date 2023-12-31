#### University: [ITMO University](https://##3itmo.ru/ru/)
#### Faculty: [FICT](https://fict.itmo.ru)
#### Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
#### Year: 2023/2024
#### Group: K34212
#### Author: Spevak Elena Aleksandrovna
#### Lab: Lab1
#### Date of create: 24.10.2023
#### Date of finished: 26.10.2023

# **Отчёт по лабораторной работе №1**
# "Установка CHR и Ansible, настройка VPN"

**Цель работы** - развертывание виртуальной машины на азе платформы Yandex Cloud с установленной системой контроля конфигурации Ansible и установка CHR в VirtualBox.

В ходе работы была развернута вирутальная машина Ubuntu 22.04 с помощью Yandex Cloud.
![scheme](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/screenshots/VM_YandexCloud.png)

У машины установлены внешний IP-адресс 158.160.44.208 и внутренний - 10.128.0.21.

В виртуальной машине быи установлены python3 (версия 3.10.12) и Ansible с помощью команд:

```
sudo apt install python3-pip
sudo pip3 install ansible
```
Следующим шагом было создание виртуальной машины CHR в VirtualBox. На официальном сайте Mikrotic был скачен образ chr-6.49.10. Был создан CHR, в котором был настроен пользователь admin.

**Настройка сервера WireGuard**

На виртуальной машине Ubuntu был поднят сервер  WireGuard для организации VPN туннеля между сервером автоматизации с установленной системой контроля Ansible и локальным CHR.
Для поднятия сервера был установлен Wireguard, сгенерирована пара ключей бля сервера и создан конфигурационный файл.
Конфигурационный файл имел вид:

```
[Interface]
Address = 10.2.0.1/24
SaveConfig = true
ListenPort = 51820
PrivateKey = [публичный ключ сервера]

[Peer]
PublicKey = [приватный ключ клиента]
AllowedIPs = 10.2.0.2/32
```
После этого был запущен сервер. Статус работы отображен на рисунке:

![Статус работы Wireguard](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/screenshots/status_wireguard.png)

**Настройка клиента WireGuard**

Чтобы поднять туннель VPN на локальном CHR был настроен Wireguard клиент. Для этого были добалвены интерфейс WireGuard, на котором был настроен адрес из конфигурационного файла на сервере(AllowedIPs), и wireguard interface peer. Так как Firewall не предполагает наличия соединения WireGuard интерфейса, на нём было прописано правило.

В итоге, CHR имеет следующийэкспорт:
![Экспорт CHR](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/screenshots/export.png)

В результате лабораторной работы была получена схема, представленная ниже.

![Схема сети](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/NP_schema_lab1.png)

Проверка связанности сервера автоматизации и локальной виртуальной машины с установленым CHR представлена на скриншотах:

![ping_ubuntu](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/screenshots/ping_ubuntu.png)

![ping_cht](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/screenshots/ping_chr.png)

**Вывод**
Был создан сервер автоматизации с установленной системой контроля конфигураций Ansible в облачном сервисе Yandex Cloud и виртуальная машина на локальном устройстве с установленной RouterOS в VirtualBox. Между ними был настроен VPN тунель с помощью протокола WireGuard.




