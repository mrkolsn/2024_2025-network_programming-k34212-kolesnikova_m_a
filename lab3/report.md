University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34212

Author: Kolesnikova Maria Alexeevna

Lab: Lab3

Date of create: 09.01.2024

Date of finished: 09.01.2024

# Лабораторная работа №3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

## Цель работы
С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

## Ход выполнения работы

### Установка NetBox
PostgreSQL нужен для работы NetBox как основная реляционная база данных. Он используется для хранения всей структурированной информации

![image_2024-12-13_18-22-27](https://github.com/user-attachments/assets/710a221d-0345-4d9d-8cbe-f0484633b7bd)


Redis в NetBox выполняет три ключевые функции: кэширование данных, асинхронная обработка задач и управление пользовательскими сессиями

![image_2024-12-13_18-25-46](https://github.com/user-attachments/assets/fb5de9f9-916c-4526-a775-dc3ca797eb16)


Клонируем репозиторий NetBox

![image_2024-12-13_18-31-46](https://github.com/user-attachments/assets/b55079bf-981a-40e7-a97a-1c32206446b2)


Создаем виртуальное окружение, устанавливаем необходимые зависимости и настраиваем конфигурационный файл configuration.py (копируем содержимое файла configuration_example.py), генерируем секретный ключ.

Инициализируем базу данных и создаем суперпользователя

Устанавливаем Gunicorn - это WSGI-сервер, который запускает Python-приложение NetBox и обрабатывает запросы, и Nginx - веб-сервер, который используется как обратный прокси для Gunicorn.
После этого запускам netbox и nginx

![Снимок экрана 2025-01-09 132539](https://github.com/user-attachments/assets/21d5b0d7-dd8b-4ae8-befd-a56461553a42)


Пробрасываем нужные порты в VirtualBox, открываем браузер и переходим по ссылке https://127.0.0.1:8000

![Снимок экрана 2025-01-09 134030](https://github.com/user-attachments/assets/cdc4f568-b1fe-4ed8-ab66-a54833d4577c)


### Добавление информации об устройствах

Во вкладке "Устройства" была внесена информация о двух роутерах R1 и R2 (роль, производитель, тип, IP-адрес)

![Снимок экрана 2025-01-09 171540](https://github.com/user-attachments/assets/abc4e3a8-8ef1-4dbb-9181-2996b1a28939)


### Сохранение данных из NetBox в файл

Файл inventory.yaml:
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
```
Роль fetch_info (roles/fetch_info/tasks/main.yaml) для сбора данных и сохранения их в файл:

![Снимок экрана 2025-01-09 180406](https://github.com/user-attachments/assets/1dd76512-5060-49de-881a-0a26866d2dcf)


Ansible плейбук playbook.yaml:

![Снимок экрана 2025-01-09 180740](https://github.com/user-attachments/assets/f4d0dee4-854d-4c45-89fe-920dac097904)


Запускаем выполнение плейбука командой
```bash
ansible-playbook -i inventory.yaml playbook.yaml
```

![Снимок экрана 2025-01-10 170721](https://github.com/user-attachments/assets/80cb8c22-9568-4b0f-ba86-0524d3bf8d3a)


Получили файл netbox_devices.json, содержащий собранную информацию об устройствах:

![Снимок экрана 2025-01-10 171950](https://github.com/user-attachments/assets/40bdc530-186e-47bd-b712-86bd7d60f625)
![Снимок экрана 2025-01-10 172000](https://github.com/user-attachments/assets/017ae352-7a0c-4caa-9582-7c1dd9ab0be0)
![Снимок экрана 2025-01-10 172013](https://github.com/user-attachments/assets/8fe3f403-2f32-4da4-ad49-742a2a805c88)


### Изменение имени устройства и добавление IP-адреса

Файл inventory.yaml:

![Снимок экрана 2025-01-10 171258](https://github.com/user-attachments/assets/7cbdc9ec-1e95-4f4c-8c83-8d9dc18d426e)


Плейбук для изменения настроек роутера в соответствии с данными из NetBox:

![Снимок экрана 2025-01-09 182342](https://github.com/user-attachments/assets/eec44fdd-1e2c-4977-92e6-92c07fa42650)


После выполнения плейбука, проверяем изменения на роутере:

Изменение имени устройства с MikroTik на R2

![Снимок экрана 2025-01-10 172213](https://github.com/user-attachments/assets/a7fb1b15-1f42-4f1c-92e1-d3a84ced5889)


Добавление IP-адреса 192.168.0.105/24

![Снимок экрана 2025-01-10 172835](https://github.com/user-attachments/assets/b939ead4-7244-4afa-9eb4-bbfdb6eba674)


### Добавление серийного номера в NetBox

Сперва собираем информацию о серийном номере устройства командой ```bash /system license print ```. Затем выполняем PATCH-запрос, чтобы обновить серийный номер устройства в Netbox.

Плейбук для добавления серийного номера в NetBox:

![Снимок экрана 2025-01-10 173157](https://github.com/user-attachments/assets/66efa84f-3be1-4f4e-8989-e5ef2284a13e)


Успешное выполнение плейбука:

![Снимок экрана 2025-01-10 173545](https://github.com/user-attachments/assets/7c0ae437-4a94-40e9-8445-2eea6ac88418)
