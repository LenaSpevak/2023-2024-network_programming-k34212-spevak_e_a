#### University: [ITMO University](https://##3itmo.ru/ru/)
#### Faculty: [FICT](https://fict.itmo.ru)
#### Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
#### Year: 2023/2024
#### Group: K34212
#### Author: Spevak Elena Aleksandrovna
#### Lab: Lab4
#### Date of create: 03.12.2023
#### Date of finished: 7.12.2023

# **Отчет по лабораторной работе №4** 

# "Базовая 'коммутация' и туннелирование используя язык программирования P4"

**Цель работы** - изучить синтаксис языка программирования P4 и выполнить 2 задания обучающих задания от Open network foundation для ознакомления на практике с P4.

**Ход работы**
 
Перед выполнением работы был склонирован репозиторий [p4lang/tutorials](https://github.com/p4lang/tutorials/tree/master) и установлен Vagrant.

После перехода в папку vm-ubuntu-20.04 была развернута тестовая среда с помощью Vagrant:```vagrant up```. В развернутой среде была установлена виртумальная машина с пользователями p4 и vagrant.  

![](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/vm_vargrant.png)

1. Задание Implementing Basic Forwarding

После авторизации была открыта папка проекта ```p4\tutorials\exercises\basic```, и в ней был дополнен файл basic.p4. В нём были дописаны функции Parser, MyIngress и Deparser. 

Данный код позволяет воссоздать процесc передачи пакетов между устройствами - роутерами. В файле прописаны типы данных полей заголовоков, сами заголовки и основные функции.  

В Parser были добавлены функции, которые парсят Ethernet и IPv4 заголовки.

![Parser](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/parser1.png)

В MyIngress происходит назначение порта, куда будет направлен пакет, обновление адресов назначения и источника, а также уменьшение time-to-live (ttl). В данной функции была добавлена проверка считваыния таблицей адреса назначения и принятия решения пересылки пакета дальше или же отклонения (вызов функций  ```ipv4_forward``` или ```drop```).

![MyIngress](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/MyIngress1.png)

![apply](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/apply_in_MyIngress1.png)

Был дополнен Deparser - выбор порядка вставки полей при сборке заголовка пакета обратно.

![Deparser](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/Deparser1.png)

Дополненный файл p4 - [basic.p4](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/files/basic.p4)

Для проверки работоспособности кода были выполнены следующие команды:

```
make run

mininet> h1 ping h2
mininet> pingall
```
Результат проверки:

![Проверка](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/testing_basic.p4.png)

2. Задание Implementing Basic Tunneling

 Для выполнения работы была открыта папка ```p4\tutorials\exercises\basic_tunneling```, у которой редактировался файл basic_tunneling.p4. В нём были дополнены функции Parser, MyIngress и Deparser.

В Parser был добавлен pars_tunnel, извлекающий заголовки myTunnel или Ipv4 из заголовка  Ethernet.

![Parser](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/Parser2.png)

В MyIngress был добавлен action myTunnel_forward, в котором устанавливается порт выхода egress port. Также была определена новая таблица myTunnel_exact, которая сопоставляет поле dst_id заголовка myTunnel. Было обновлено action.

![MyIngress](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/MyIngress2.png)

В Deparser были прописан порядок вставки полей при сборке заголовка пакета обратно:  ethernet, myTunnel, ipv4.

![Deparser](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/Deparser2.png)

Дополненный файл p4 - [basic_tunnel.p4](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/files/basic_tunnel.p4)

После сохранения изменений была проверена работоспособность кода.

В терминале были запущены команды 

```
make run

mininet> xterm h1  h2
```

Были открыты два терминала устройств: h1 и h2. Во втором была прописана команда ```./receive.py ``` для запуска сервера. На первом был проведен тест ```./send.py 10.0.2.2 "P4 is cool"```. В результате пакет с сообщением "P4 is cool" появился и  терминале устройства h2. 

![test](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/check1.png)

В пакете содержатся заголовки Ethernet, Ip, TCP и само сообщение. В следующей проверке был установлен id назначения (dst_id 2). Пакет с сообщением дошел до h2.

![checking_tunneling](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/check2.png)

После этого была произведена проверка с указанием другого адреса (10.0.3.3), но прежнего id назначения (dst_id 2). Несмотря на то, что указан адрес другого устройства, пакет всё равно отобразился в терминале h2,так как был указан его id.

![checking_tunneling2](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab4/screenshots/check3.png)

**Вывод**

Был изучен синтаксис языка программирования P4 и выполнены 2 задания обучающих задания от Open network foundation: Implementing Basic Forwarding и Implementing Basic Tunneling, для ознакомления на практике с P4.



