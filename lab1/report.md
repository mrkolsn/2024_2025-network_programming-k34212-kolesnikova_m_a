University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34212

Author: Kolesnikova Maria Alexeevna

Lab: Lab1

Date of create: 29.09.2024

Date of finished: 30.09.2024

# Лабораторная работа №1 "Установка CHR и Ansible, настройка VPN"

## Описание
Данная работа предусматривает обучение развертыванию виртуальных машин (VM) и системы контроля конфигураций Ansible а также организации собственных VPN серверов.

## Цель работы
Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox.

## Ход выполнения работы
### Настройка сервера
1. Создание виртуальной машины Ubuntu
   
   ![photo_2024-09-30_17-29-03](https://github.com/user-attachments/assets/9bf947ad-d833-472f-8c3f-da5331ec3661)

2. Установка python3 и Ansible
   
   ![Снимок экрана 2024-09-27 184015](https://github.com/user-attachments/assets/bdf89fce-ebd4-479d-a537-34034b031eac)

3. Установка Wireguard
   
   ![Снимок экрана 2024-09-30 175048](https://github.com/user-attachments/assets/1cb0dd6b-2b1b-41f7-82af-5f71d101ee33)
   
   Были сгенерированы приватный и публичный ключи. С полученным публичным ключем клиента был прописан конфигурационный файл wg0.conf
   
   ![Снимок экрана 2024-09-30 180922](https://github.com/user-attachments/assets/8707772c-1699-4014-a227-a828519bd42c)
   
   Запуск службы
   
   ![Снимок экрана 2024-09-30 164637](https://github.com/user-attachments/assets/edea3218-b85d-4ae0-b392-6bef56592a83)
   
### Настройка клиента
4. Была создана другая виртуальная машина
   
   ![Снимок экрана 2024-09-27 181436](https://github.com/user-attachments/assets/16f6cbb6-9816-403f-ab62-8a550aaa3c98)
   
   На ней был установлен CHR
   
   ![Снимок экрана 2024-09-30 181448](https://github.com/user-attachments/assets/e39096f3-b261-45bf-bc39-f08c4f71e3df)
   
5. Для настройки клиента было выполнено создание интерфейса WireGuard через WinBox
   
   ![Снимок экрана 2024-09-30 181911](https://github.com/user-attachments/assets/0ed144b5-6078-4499-9bb7-2cfaa6a2ce77)
   
   Добавление информации о VPN-севере
   
   ![Снимок экрана 2024-09-30 182053](https://github.com/user-attachments/assets/bb658f08-caf9-424d-8e6b-07e044574edf)
   
   Созданному интерфейсу был присвоен ip адрес 10.0.0.2
   
   ![Снимок экрана 2024-09-30 182243](https://github.com/user-attachments/assets/b6f9b3fe-998c-427d-89fe-d8bba7904247)

   
### Тестирование
6. Доступность сервера
   
   ![Снимок экрана 2024-09-30 182735](https://github.com/user-attachments/assets/cbac415e-d89a-44c7-b5e1-8365abcdd40a)

7. Доступность клиента
   
      ![Снимок экрана 2024-09-30 182649](https://github.com/user-attachments/assets/65653e8e-57f6-4587-a41e-312c5c034d3e)

