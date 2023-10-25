#### University: [ITMO University](https://##3itmo.ru/ru/)
#### Faculty: [FICT](https://fict.itmo.ru)
#### Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
#### Year: 2023/2024
#### Group: K33212
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

На виртуальной машине Ubuntu был поднят сервер  Wireguard для организации VPN туннеля между сервером автоматизации с установленной системой контроля Ansible и локаьным CHR.

Чтобы поднять туннель VPN на локальном CHR был настроен Wireguard клиент.

В результате лабораторной работы была получена схема, представленная ниже.
![Схема сети](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab1/NP_lab1.drawio.png)


Проверка связанности сервера автоматизации и локальной виртуальной машины с установленым CHR представлена на скриншоте:
![scheme]()

**Вывод**
Был создан сервер автоматизации с установленной системой контроля конфигураций Ansible в облачном сервисе Yandex Cloud и виртуальная машина на локальном устройстве с установленной RouterOS в VirtualBox. Между ними был настроен VPN тунель с помощью протокола Wireguard.




