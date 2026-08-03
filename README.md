# Network-Design-for-Superstore
# Проект: Проектирование и настройка ИТ-инфраструктуры компании "Суперстор"

## Описание проекта
В рамках курса Яндекс.Практикум я спроектировал и полностью настроил ИТ-инфраструктуру для распределённой компании "Суперстор". Проект включает в себя два офиса (головной офис и даркстор), связанные через выделенный канал связи.

**Цель проекта:** Создать отказоустойчивую, сегментированную и безопасную сетевую инфраструктуру, развернуть все необходимые сервисы для работы сотрудников и обеспечить публикацию веб-приложения в интернет.

## Стек технологий
- **Сетевое оборудование:** Cisco IOS (маршрутизаторы, коммутаторы)
- **Сетевые технологии:** VLAN, Trunk (802.1Q), Subinterfaces, Static Routing, NAT/PAT, DNAT (Port Forwarding)
- **Операционные системы:** Linux (Ubuntu/Debian)
- **Сервисы и службы:** NTP, DNS (dnsmasq), DHCP, Postfix (почта), vsftpd (FTP), Nginx (веб-сервер)
- **Автоматизация:** Ansible
- **Эмуляция:** EVE-NG (сеть), VirtualBox/VMware (серверы)

## Схема сети
<img width="1999" height="1064" alt="image1_1746525626" src="https://github.com/user-attachments/assets/d8af46b2-6011-4bda-9008-2f8c4cb0fa57" />

## Реализованные задачи

### Часть 1: Дизайн и конфигурация сети

#### 1. VLAN и базовая связность
- Созданы все необходимые VLAN: **IT (10)**, **Management (20)**, **Logistics (30)**, **Storage (40)**, **HQ-Servers (50)**, **DS-Servers (60)**.
- Настроены trunk-порты (802.1Q) между коммутаторами и маршрутизаторами для передачи трафика VLAN.
- Серверам назначены статические IP-адреса из соответствующих подсетей.
- Настроены порты доступа для пользовательских устройств с комментариями (описаниями).
- **Результат:** Успешная проверка связности `ping` между устройствами в одной VLAN.

## **VLAN и базовая связность PCL-3 → PCL-4** 
<img width="665" height="225" alt="Тест 1 1" src="https://github.com/user-attachments/assets/7defc753-8bd1-4f8e-9551-31692dbab5e1" />

## **VLAN и базовая связность PCS-1 → PCS-2**
<img width="622" height="164" alt="Тест 1 4" src="https://github.com/user-attachments/assets/16c8c079-9814-4e32-b2d5-67ac869e8745" />

## **Доступ в интернет и NAT PCL-1 → шлюз**
<img width="652" height="209" alt="Тест 2 1 1" src="https://github.com/user-attachments/assets/fdc2e0b9-ff6b-4ed6-a955-48c6887a969a" />

## **Доступ в интернет и NAT ping с R-Darkstore до ISP и таблица маршрутизации R-Darkstore**
<img width="554" height="90" alt="Проверка 1" src="https://github.com/user-attachments/assets/dc3e8809-6652-4d20-9652-33ff2e286b46" />
<img width="831" height="426" alt="Проверка 2" src="https://github.com/user-attachments/assets/eecbdd87-07bf-4c7d-8ca3-2528670206c0" />





#### 2. Межсетевая маршрутизация
- На маршрутизаторах `R-HQ` и `R-Darkstore` настроены субинтерфейсы для всех VLAN.
- Каждый субинтерфейс получил IP-адрес, выступающий в роли шлюза по умолчанию для своей подсети.
- **Результат:** Устройства из разных VLAN успешно взаимодействуют друг с другом.

<img width="633" height="161" alt="Проверка 1" src="https://github.com/user-attachments/assets/6702e825-f8c7-49a8-9fc3-ccd540b5d5e3" />
<img width="639" height="164" alt="Проверка 2" src="https://github.com/user-attachments/assets/a99efe0a-5f5a-4b1a-9e30-542ab4e7cf36" />




#### 3. Настройка DHCP
- На маршрутизаторах `R-HQ` и `R-Darkstore` настроены DHCP-пулы для автоматической раздачи IP-адресов пользователям в VLAN 10, 20, 30, 40.
- Статические адреса с клиентских машин удалены.
- **Результат:** Все пользовательские устройства получают настройки сети (IP, маску, шлюз, DNS) автоматически.
- 
<img width="859" height="113" alt="Проверка 1 1" src="https://github.com/user-attachments/assets/9abdc1b7-7458-42cd-bc33-25f9002f2b69" />
<img width="696" height="67" alt="Проверка 2 3" src="https://github.com/user-attachments/assets/8b1501b2-1c68-4753-8e53-3c28d3af4cc9" />



#### 4. Доступ в интернет и NAT
- Настроены стыки с провайдерами: интерфейсам `R-HQ` и `R-Darkstore` назначены публичные IP-адреса (`55.55.55.101/30` и `55.55.55.105/30`).
- Настроены маршруты по умолчанию на обоих маршрутизаторах.
- Настроен Source NAT (PAT) для всего исходящего трафика из локальных сетей.
- **Результат:** Все устройства в локальной сети имеют доступ в интернет.
<img width="640" height="96" alt="Проверка 1" src="https://github.com/user-attachments/assets/5cd79208-4972-49e9-a812-e785de8ada60" />
<img width="852" height="329" alt="Проверка 2" src="https://github.com/user-attachments/assets/23f739f1-5dc3-4624-a204-60cd1fe16301" />



#### 5. Публикация веб-сервера (DNAT)
- На маршрутизаторе `R-HQ` настроен Static NAT (Port Forwarding) для веб-сервера (`10.10.5.60`).
- Порт TCP 80 внутреннего сервера опубликован на внешнем IP-адресе через порт 8080.
- **Результат:** Веб-сервер доступен из интернета по адресу `55.55.55.101:8080`.
<img width="591" height="209" alt="Проверка 1" src="https://github.com/user-attachments/assets/81819f1f-9b59-4abe-a24a-3b89ed18ee40" />
<img width="577" height="160" alt="Проверка 2" src="https://github.com/user-attachments/assets/475af1c7-9b4b-4394-bcef-3643e790c403" />
<img width="561" height="161" alt="Проверка 3" src="https://github.com/user-attachments/assets/7ef383c6-7ae4-459d-892e-6eab461e88e8" />
<img width="658" height="288" alt="Проверка 4" src="https://github.com/user-attachments/assets/c5477ff1-eeaf-41f3-a050-d9d1f20ae4bb" />



#### 6. Связь между офисами
- Настроена статическая маршрутизация через провайдера WAN.
- На маршрутизаторах `R-HQ`, `R-Darkstore` и `WAN` добавлены маршруты до внутренних подсетей другой локации.
- **Результат:** Устройства из головного офиса и даркстора успешно обмениваются данными.
<img width="922" height="420" alt="R-Darkstore" src="https://github.com/user-attachments/assets/30f3cf70-cc71-455a-8adb-a3c15b260ae1" />
<img width="877" height="416" alt="R-HQ" src="https://github.com/user-attachments/assets/679a650d-14ff-4b8a-a86f-b905abf37bab" />
<img width="820" height="354" alt="WAN" src="https://github.com/user-attachments/assets/fd7b60ca-b5a3-462d-9882-333499024ce8" />
<img width="632" height="161" alt="Проверка 1 1" src="https://github.com/user-attachments/assets/550f9489-6d62-4a17-ade0-f44da26490d3" />


---

### Часть 2: Настройка базовых и прикладных сервисов

#### 1. Настройка NTP и DNS
- На **Хосте 1** развернут NTP-сервер (chrony) с временной зоной Москвы (UTC+3).
- На **Хосте 1** развернут DNS-сервер (dnsmasq) для домена `practicumsuperstore.ru`.
- Созданы А-записи для ключевых сервисов: `WEB`, `AD`, `FS`, `mail`.
- **Результат:** Все хосты синхронизируют время и резолвят внутренние доменные имена.
<img width="920" height="481" alt="Тест 1" src="https://github.com/user-attachments/assets/9c39199a-aa1f-444d-aeca-f60874c15f68" />
<img width="645" height="131" alt="Тест 2" src="https://github.com/user-attachments/assets/92c38302-b2d6-4f3d-b30e-a6b5979bcfdf" />
<img width="646" height="226" alt="Тест 3 1" src="https://github.com/user-attachments/assets/040784f8-9a01-4199-9897-2622b6b2fc66" />
<img width="553" height="220" alt="Тест 3 2" src="https://github.com/user-attachments/assets/d444b708-9b08-4abb-9875-fcdf4012523c" />
<img width="550" height="138" alt="Тест 4" src="https://github.com/user-attachments/assets/0d3d23b7-5482-4115-8ecc-a10dbe0a1a39" />
<img width="496" height="177" alt="Тест 5 1" src="https://github.com/user-attachments/assets/417f5021-ded4-4081-9c06-168a6cd03de1" />
<img width="586" height="181" alt="Тест 5 2" src="https://github.com/user-attachments/assets/fe81234e-f557-476f-be00-81aa70bbb2fa" />


#### 2. Автоматизация с Ansible
- На **Хосте 2** развернут Ansible-контроллер.
- Настроено SSH-взаимодействие между контроллером и управляемым **Хостом 3**.
- Написаны и применены плейбуки:
  - `setup_services.yml` — настройка DNS и NTP на клиенте.
  - `create_user.yml` — создание пользователя `backup-user`.
  - `configure_backup.yml` — настройка резервного копирования через cron.
- **Результат:** Управление конфигурацией клиента полностью автоматизировано.
<img width="517" height="129" alt="Тест 1 1" src="https://github.com/user-attachments/assets/78508c09-32d2-425d-91ae-3742cfa10980" />
<img width="517" height="84" alt="Тест 1 2" src="https://github.com/user-attachments/assets/26442a69-572e-4884-8ed8-b635db14697a" />
<img width="541" height="212" alt="Тест 2" src="https://github.com/user-attachments/assets/e0a47c93-fbb9-42e1-b369-5d76e63e7843" />
<img width="642" height="96" alt="Тест 3" src="https://github.com/user-attachments/assets/b9c9c248-470b-4957-8764-16491736a2cf" />
<img width="643" height="209" alt="Тест 4 1" src="https://github.com/user-attachments/assets/4f55d2b3-e937-449d-b8ec-43c2abc3a347" />
<img width="576" height="120" alt="Тест 4 2" src="https://github.com/user-attachments/assets/f7895c97-8c47-42c8-98ed-d33b32739f0d" />


#### 3. Развертывание почтовой инфраструктуры (Postfix)
- На **Хосте 1** установлен и настроен почтовый сервер Postfix.
- В DNS добавлена MX-запись для `mail.practicumsuperstore.ru`.
- На **Хосте 3** настроен почтовый клиент.
- Отправлено и получено тестовое письмо для пользователя `ubuntu`.
- **Результат:** Почтовый сервер успешно принимает и доставляет письма.
<img width="908" height="245" alt="Тест 1" src="https://github.com/user-attachments/assets/74b7f1cd-c0bb-421d-a5ca-51f789d87f05" />
<img width="621" height="69" alt="Тест 2 1" src="https://github.com/user-attachments/assets/ea40f43a-21e0-4bab-99fb-f88d279d46d9" />
<img width="607" height="82" alt="Тест 2 2" src="https://github.com/user-attachments/assets/aef6dcc8-6249-4324-8a06-904700416d47" />
<img width="680" height="231" alt="Тест 3" src="https://github.com/user-attachments/assets/3ddccf3e-fa7e-4117-8e63-07593c8e6d3b" />
<img width="791" height="67" alt="Тест 4" src="https://github.com/user-attachments/assets/3f8cf04c-b619-4d1f-a4ec-18e500747c77" />
<img width="774" height="183" alt="Тест 5" src="https://github.com/user-attachments/assets/fd68c60d-430b-4062-9ddf-2ccb6fa4fda3" />

#### 4. Настройка файлового обмена (FTP)
- На **Хосте 2** установлен FTP-сервер `vsftpd`.
- Создан пользователь `ftpuser` и настроена директория `/home/ftpuser/ftp/files`.
- Настроены параметры для локальных пользователей и возможности записи.
- **Результат:** Успешная загрузка и скачивание тестового файла с FTP-сервера.
<img width="757" height="511" alt="Тест 1" src="https://github.com/user-attachments/assets/dcfee453-90f6-41b8-8739-16098dbf642a" />
<img width="665" height="225" alt="Тест 2" src="https://github.com/user-attachments/assets/89a445ca-ceba-418a-a4c2-10b77adedf3c" />

#### 5. Развертывание веб-сервера (Nginx)
- На **Хосте 2** установлен и настроен веб-сервер Nginx.
- Настроены кастомные страницы ошибок (404, 500).
- Настроена базовая аутентификация (HTTP Auth) для директории `/secure`.
- **Результат:** Веб-сервер доступен, работает аутентификация.
<img width="1883" height="1022" alt="Тест 1" src="https://github.com/user-attachments/assets/85c04e41-cedc-4a7b-9436-658e3c16dcda" />
<img width="643" height="64" alt="Тест 2" src="https://github.com/user-attachments/assets/42c35e1b-a408-4b85-9d50-0505e6123c63" />


---

## Выводы и навыки, полученные в ходе проекта
- **Сетевой инжиниринг:** На практике освоил технологии VLAN, Trunk, межсетевой маршрутизации и NAT на оборудовании Cisco.
- **Администрирование Linux:** Закрепил навыки настройки сетевых и прикладных сервисов (DNS, NTP, Postfix, Nginx, vsftpd).
- **Инфраструктура как код:** Получил практический опыт работы с Ansible для автоматизации рутинных задач.
- **Комплексное мышление:** Научился видеть полную картину и обеспечивать взаимодействие сетевого и серверного уровня.

Проект является частью моего портфолио и демонстрирует мою готовность к работе в роли **Инженера по информационной безопасности / Системного администратора.
