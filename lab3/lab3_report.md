#### University: [ITMO University](https://##3itmo.ru/ru/)
#### Faculty: [FICT](https://fict.itmo.ru)
#### Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
#### Year: 2023/2024
#### Group: K34212
#### Author: Spevak Elena Aleksandrovna
#### Lab: Lab3
#### Date of create: 18.11.2023
#### Date of finished: 02.12.2023

# **Отчёт по лабыораторной работе №3**
# "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

**Цель работы** - с помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

**Ход работы**

1. Поднятие NetBox
  
  На виртуальной машине в YandexCloud был поднят NetBox. Для этого последовательно были настроены база данных PostrgreSQL, Redis, NetBox, Gunicorn и HTTP сервер nginx.

 - Настройка базы данных PostgreSQL:

  Были установлены пакеты postrgresql. Версия установленной базы данных должна быть не ниже 12.

  ```
  sudo apt update
  sudo apt install -y postgresql
  ```

  Была создана база данных и настроен пользователь для допуска к ней. Код для создания базы данных представлен ниже.

  ```
    CREATE DATABASE netbox;
    CREATE USER <username> WITH PASSWORD '<password>';
    ALTER DATABASE netbox OWNER TO netbox;
    -- the next two commands are needed on PostgreSQL 15 and later
    \connect netbox;
    GRANT CREATE ON SCHEMA public TO netbox;
  ```
  Пользователь и пароль были установлены свои.
  
  Был проверен статус системы:

  ```
  psql --username netbox --password --host localhost netbox
  ```

  ![PostreSQL status](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/psql_status.png)

 - Настройка хранилища данных Redis
  
  NetBox использует Redis для кеширования и орагнизации очередей.

  Были установлены пакеты. Версия не должна быть ниже 4.1. 

  ```
  sudo apt install -y redis-server
  ```

  Была проверен статус службы с помощью команды  
  ```
  redis-cli ping
  ```

  ![проверка службы redis](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/redis_cheking.png)

 - Установка необходимых пакетов для NetBox
  
  Перед установкой самого NetBox были установлены все необходимые системные пакеты Python (система управления пакетами, окружение, пакет с файлами заголовков для API и тд.). Вресия установленного Python не должна быть ниже 3.8.

  ```
  sudo apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev
  ```

  Была создана директория, куда была скллонирована ветка из репозитория NetBox.

  ```
  sudo mkdir -p /opt/netbox/
  cd /opt/netbox/
  sudo git clone -b master --depth 1 https://github.com/netbox-community/netbox.git .
  ```

  Был создан системный пользователь netbox  и настроены его права.

  ```
  sudo adduser --system --group netbox
  sudo chown --recursive netbox /opt/netbox/netbox/media/
  sudo chown --recursive netbox /opt/netbox/netbox/reports/
  sudo chown --recursive netbox /opt/netbox/netbox/scripts/
  ```
  
  Был создан конфигурационный файл - копия cnfiguratiom_example.py для хранения локальных параметров конфигурации.

  ```
  cd /opt/netbox/netbox/netbox/
  sudo cp configuration_example.py configuration.py
  ```
  В этом файле были прописаны список хостов(ALLOWED_HOSTS), параметры базы данных(имя пользователя, пароль и т.д.), параметры Redis, случайно сгенерированный ключ(SECRET_KEY).

  Код для генерирования ключа

  ```
  python3 ../generate_secret_key.py
  ```

  Файл имел вид:

  ```
  ALLOWED_HOSTS = ['*']
  DATABASE = {
     'NAME': 'netbox',               # Database name
     'USER': 'netbox',               # PostgreSQL username
     'PASSWORD': 'пароль',           # PostgreSQL password
     'HOST': 'localhost',            # Database server
     'PORT': '',                     # Database port (leave blank for default)
     'CONN_MAX_AGE': 300,            # Max database connection age (seconds)
  }

  SECRET_KEY = 'сгенерированный_секретный_ключ'
  ```
  Был заупщен скрипт для создания виртуальной среды Python, установки всех необходимых пакетов Python, запуска миграции схемы базы данных:

  ```
  sudo /opt/netbox/upgrade.sh
  ```
  
  После этого был создан супер пользователь - администратор для досупа к NetBox. Была активирована вирутальная среда Python.

  ```
  source /opt/netbox/venv/bin/activate
  cd /opt/netbox/netbox
  python3 manage.py createsuperuser
  ```

 - Настройка Gunicorn
  
  NetBox работает как WSGI-приложение за HTTP-сервером, поэтому было необзодимо настроить Gunicorn. 

  Был скопирован конфигурационный файл, в котором уже были необходимые настройки, и другие необходимые файлы. После этого был перезапущен демон systemd

  ```
  sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
  sudo cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
  sudo systemctl daemon-reload
  ```

  Были запущены сервисы netbox и netbox-rq и включен их запуск во время загрузки. После запуска был проверен их статус:

  ```
  sudo systemctl start netbox netbox-rq
  sudo systemctl enable netbox netbox-rq
  ```
  ![Статуст netbox](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/netbox_status.jpg)

 - Настройка HTTP сервера
  
  Был выбран nginx.

  Были произведены его установка и копирование конфигурационного файла, созданного NetBox.

  ```
  sudo apt install -y nginx
  sudo cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
  ```

  Был заменен созданный по умолчанию nginx конфигурационный файл на файл, созданный выше:

  ```
  sudo rm /etc/nginx/sites-enabled/default
  sudo ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox
  ```

  После этого сервис был перезапущен
  
  ```
  sudo systemctl restart nginx
  ```

  Приложение NetBox стало доступно как веб-приложение по ip-адресу виртуальной машины: https://158.160.48.189

  ![NetBox приложение](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/netbox_interface.jpg)

2. Заполнение информации об устройствах в NetBox 

  В NetBox были добавлены устройства - CHR1 и CHR2,а в них - данные об интерфесах и IP-адреса. Так же были добавлены такие записи как, Site(NetProgLabs - физическое расположение устройств), роль (роутер), производитель (MikroTik)и тип (RouterOsv7), которые указывались в описании устройств.

 ![Интерфейсы](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/devices_interfaces.png)

 ![IP-адреса](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/ip_addresses.png)

 ![Устройства](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/netbox_devices.jpg)

 
3. Сохранение данных из NetBox в отдельный файл

  Для сбора информации из NetBox был использован модуль netbox.netbox.nb_inventory в Ansible. Для этого был создан файл netbox_inventory.yml с следующим содержанием:

  ```
  ---
  plugin: netbox.netbox.nb_inventory
  api_endpoint: https://158.160.48.189
  token: токен
  validate_certs: False
  config_context: False
  group_by:
    - device_roles
  interfaces: 'True'
  ```

  В нём используется API токен, созданый в приложении NetBox.

  Информация об устройствах была сохранена в [inventory-файл](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/files/nb_inventory.yml).

  ```
  ansible-inventory -v --list -i netbox_inventory.yml > nb_inventory.yml
  ```

4. Настройка двух CHR по сценарию
  
  Для изменения имени устройств и добавления IP-адресов на основе [файла](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/files/nb_inventory.yml) был написан сценарий: 

  ```
  - name: Routers Configuration
    hosts: device_roles_router
    tasks:
      - name: Set Devices Name
        community.routeros.command:
          commands:
            - /system identity set name="{{interfaces[0].device.name}}"
      - name: Set additional IP
        community.routeros.command:
          commands:
          - /interface bridge add name="{{interfaces[1].display}}"
          - /ip address add address="{{interfaces[1].ip_addresses[0].address}}" interface="{{interfaces[1].display}}"
  ```
  После его выполнения был получен следующий результат

  ![Выполнение сценария](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/changing.jpg)

  ![Измененные имена](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/changing_names.jpg)

5. Написание сценария для сбора серийного номера устройства и его вноса в NetBox
   
  Был написан следующий сценарий:

  ```
  - name: Get Serial Numbers To NetBox
  hosts: device_roles_router
  tasks:
    - name: Get Serial Number
      community.routeros.command:
        commands:
          - /system license print
      register: license_print
    - name: Get Name
      community.routeros.command:
        commands:
          - /system identity print
      register: identity_print
    - name: Add Serial Number to Netbox
      netbox_device:
        netbox_url: https://158.160.48.189
        netbox_token: токен
        data:
          name: "{{identity_print.stdout_lines[0][0].split(' ').1}}"
          serial: "{{license_print.stdout_lines[0][0].split(' ').1}}"
        state: present
        validate_certs: False
  ```

  После его выполнения был получен следующий результат:

  ![Выполнение сценария](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/getting_serial_num.jpg)

  ![Серийтный номер перовго устрйоства](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/serial_num_1.jpg)

  ![Серийтный номер перовго устрйоства](https://github.com/LenaSpevak/2023-2024-network_programming-k34212-spevak_e_a/blob/main/lab3/screenshots/serial_num_2.jpg)


**Вывод**

С помощью Ansible и Netbox была собрана вся возможная информация об устройствах CHR1 и CHR2 и сохранена в отдельном файле nb_inventory.yml, был создан сценарий для смены имен устройств и добавления IP. Также были собраны серийные номера устройств и внесены в NetBox.
