#### University: [ITMO University](https://##3itmo.ru/ru/)
#### Faculty: [FICT](https://fict.itmo.ru)
#### Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
#### Year: 2023/2024
#### Group: K34212
#### Author: Spevak Elena Aleksandrovna
#### Lab: Lab2
#### Date of create: 4.11.2023
#### Date of finished:

# **Отчёт по лабораторной работе №2**
# "Развертывание дополнительного CHR, первый сценарий Ansible"

**Цель работы** - с помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

**Ход работы**

- Добавление и настройка второго CHR

В ходе работы  в  программном обеспечении VirtualBox была создана вторая виртуальная машина - CHR2. В ней была произведена настройка пользоватля admin и клиента WireGuard для поднятия  VPN соединения.

Для этого была сгенерирована вторая пара ключей при помощи команды:

```
wg genkey | sudo tee /etc/wireguard/wg0-client2-private.key | wg pubkey | sudo tee /etc/wireguard/wg0-client2-public.key
```

На сервере в конфигурационный файл /etc/wireguard/wg0.conf был добавлен второй пир:

```
[Interface]
Address = 10.2.0.1/24
ListenPort = 51820
PrivateKey = [приватный ключ сервера]

[Peer]
PublicKey = [публичный ключ 1 клиента]
AllowedIPs = 10.2.0.2/32

[Peer]
PublicKey = [публичный ключ 2 клиента]
AllowedIPs = 10.2.0.3/32
```

На CHR2 был добавлен интерфейс wireguard1 и назначен его ip-адрес, настроен пир и добавлено правило в firewall (разрешен трафик WireGuard).

```
/interface wireguard add listen-port=51820 mtu=1420 name=wireguard1
/interface wireguard peers add allowed-address=10.2.0.1/24 endpoint-address=158.160.53.43
endpoint-port=51820 interface=wireguard1 persistent-keepalive=10s
public-key="публичный ключ сервера"
/ip address add address=10.2.0.3/24 interface=wireguard1 network=10.2.0.0
/ip firewall filter add action=accept chain=input dst-port=51820 int-interface=wireguard1
protocol=udp
```
Проверка связности виртуальных машин:

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/pingCHR2.jpg)

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/pingCHR1.jpg)

CHR готовы к настройке.

- Настройка виртуальных машин с Ansible

При помощи Ansible одновременна на двух CHR были настроены:
 - логин/пароль
 - NTP Client
 - OSPF с указанием Router ID
 
Для этого были созданы:
 
1. [inventory файл](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/files/hosts.yml)
2. [playbook](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/files/lab2.yml)

**host.yml**

```
[CHRs]
CHR1 ansible_host=10.2.0.2 router_id=1.1.1.1
CHR2 ansible_host=10.2.0.3 router_id=2.2.2.2

[CHRs:vars]
ansible_user=admin
ansible_password=admin
ansible_connection=network_cli
ansible_network_os=routeros
```
В этому файле перечисляются устройства, на которых будет производиться настройка. В нашем случае это CHR1 и CHR2. В добавок в данном файле прописываются переменные, которые при настройке будет подтягивать playbook.

**playbook**

```
---
- name: "playbook for lab2"
- hosts: CHR1, CHR2
  vars:
    host_vars:
      CHR1:
        router_ip: 1.1.1.1
      CHR2:
        router_ip: 2.2.2.2

 tasks:
    - name: Set User&Password
      community.routeros.command:
        commands: "user add name=user1 password=user1 group=full"

    - name: Set NTP
      community.routeros.command:
        commands: "system ntp client set enabled=yes servers=8.8.8.8"

    - name: Set OSPF
      community.routeros.command:
        commands:
          - /interface bridge add name=Lo
          - /ip address add address="{{ router_id }}"/32 interface=Lo
          - /routing ospf instance add name=v2inst version=2 router-id="{{ router_id }}"
          - /routing ospf area add name=backbone_v2 area-id=0.0.0.0 instance=v2inst
          - /routing ospf interface-template add network=0.0.0.0/0 area=backbone_v2

    - name: Collect OSPF information
      community.routeros.command:
        commands: "/routing ospf neighbor print"
      register: ospf_info

    - name: Get OSPF information
      debug:
        msg: "{{ ospf_info }}"

    - name: Collect config
      community.routeros.facts:
        gather_subset:
          - config
      register: config_info

    - name: Get config
      debug:
        msg: "{{ config_info }}"
...
```

Этот файл содержит в себе сценарии действий настройки. В файле указано, на каких устройствах будет производиться настройка и перечислены задания, содержащие команды настройки.

После завершения настройки устройств были собранны данные по OSPF топологии и полные конфиги устройств. За это отвечают последние task-ов в playbook.

- Сбор конфигов и данных по OSPF

Собранные данные по OSPF:

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/msg_ospf.jpg)

Конфиги после настройки устройств представлены ниже:

- CHR1:

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/CHR1_config1.jpg)

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/CHR1_config2.jpg)

- CHR2:

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/CHR2_config.jpg)

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/CHR2_config2.jpg)

Полный конфиг с машины выглядит следующим образом:

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab2/screenshots/config.jpg)

**Вывод**

В данной лабораторной работе была создана вторая виртуальная машина CHR2, с помощью Ansible на CHR1 и CHR2 были настроены логин/пароль, NTP Client, OSPF. Были собраны конфигурационные данный по машинах и данные по OSPF.

