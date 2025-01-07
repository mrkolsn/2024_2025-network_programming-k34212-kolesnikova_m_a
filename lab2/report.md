University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34212

Author: Kolesnikova Maria Alexeevna

Lab: Lab2

Date of create: 18.12.2024

Date of finished: 20.12.2024

# Лабораторная работа №2 "Развертывание дополнительного CHR, первый сценарий Ansible"

## Цель работы
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

## Ход выполнения работы

1. В VirtualBox был создан второй CHR
 
![Снимок экрана 2025-01-07 213223](https://github.com/user-attachments/assets/2554cc72-63cd-48cc-828d-3eb87c944baa)

2. На созданном роутере CHR2 был настроен Wireguard интерфейс

![Снимок экрана 2025-01-07 214353](https://github.com/user-attachments/assets/aa8d355e-7461-47aa-b074-885356a60bbe)


3. На сервере в файл wg0.conf была добавлена информация о новом клиенте с адресом 10.0.0.3. Проверка доступности сервера со второго CHR:

![Снимок экрана 2024-11-26 234321](https://github.com/user-attachments/assets/ac2bc924-cee9-4051-a5cb-54328df4bf7e)


4. Был создан файл inventory.yaml, содержащий информацию об узлах, на которых следует производить настройку

![Снимок экрана 2025-01-07 220011](https://github.com/user-attachments/assets/5edbf852-b6bb-4490-9e23-7f6e7f0d6486)


5. Для настройки устройств был создан Ansible плейбук playbook.yaml:

```
---
    - name: Setup MikroTik CHR
      hosts: chr_routers
      gather_facts: no
      tasks:
        - name: Set admin login credentials
          community.routeros.command:
            commands:
              - /user set admin password="QWERTY"
          register: user_verify
          ignore_errors: no
    
        - name: Configure NTP Client
          community.routeros.command:
            commands:
              - /system ntp client set enabled=yes mode=unicast
              - /system ntp client servers add address=216.239.35.4
              - /system ntp client servers add address=216.239.35.8  # google NTP servers
              - /system ntp client print
          register: ntp_result
          ignore_errors: no
    
        - name: Configure OSPF
          community.routeros.command:
            commands:
              - /interface bridge add name=loopback
              - /ip address add address={{ chr_id }} interface=loopback
              - /routing ospf instance add disabled=no name=default
              - /routing ospf instance set default router-id={{ chr_id }}
              - /routing ospf area add instance=default name=backbone
              - /routing ospf interface-template add area=backbone interfaces=ether1 type=ptp
          vars:
            chr_id: "{{ '1.1.1.1' if ansible_host == '10.0.0.2' else '2.2.2.2' }}"
    
        - name: Gather OSPF topology
          community.routeros.command:
            commands:
              - /routing ospf neighbor print detail
          register: ospf_topology
    
        - name: Print OSPF topology
          debug:
            var: ospf_topology.stdout_lines
    
        - name: Gather full config
          community.routeros.command:
            commands:
              - /export
          register: full_config
    
        - name: Print full config
          debug:
            var: full_config.stdout_lines
```



6. Выполнение плейбука

![image_2024-12-13_17-37-04](https://github.com/user-attachments/assets/9ae8bb9e-9da8-4397-9711-e31ef99ec601)


7. Данные по OSPF топологии

```bash
TASK [Print OSPF topology] ******************************************************************************************************************************************************************
ok: [chr_2] => {
    "ospf_topology.stdout_lines": [
        [
            "Flags: V - virtual; D - dynamic ",
            " 0  D instance=default area=backbone address=172.20.10.4 router-id=1.1.1.1 ",
            "      state=\"Full\" state-changes=6 adjacency=4m5s timeout=36s"
        ]
    ]
}
ok: [chr_1] => {
    "ospf_topology.stdout_lines": [
        [
            "Flags: V - virtual; D - dynamic ",
            " 0  D instance=default area=backbone address=172.20.10.6 router-id=2.2.2.2 ",
            "      state=\"Full\" state-changes=5 adjacency=4m5s timeout=35s"
        ]
    ]
}
```

8. Полный конфиг устройств

```bash
TASK [Print full config] ********************************************************************************************************************************************************************
ok: [chr_1] => {
    "full_config.stdout_lines": [
        [
            "# 2024-12-13 14:36:32 by RouterOS 7.16.1",
            "# software id = ",
            "#",
            "/interface bridge",
            "add name=loopback",
            "/interface wireguard",
            "add listen-port=51820 mtu=1420 name=wg0",
            "/routing ospf instance",
            "add disabled=no name=default router-id=1.1.1.1",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.0.0.0/24 endpoint-address=172.20.10.5 endpoint-port=\\",
            "    51820 interface=wg0 name=peer2 public-key=\\",
            "    \"TqqKO9e9OMq34Nk2zESioaVDMLyrHUL+9m3rWM82WWI=\"",
            "/ip address",
            "add address=10.0.0.2/24 interface=wg0 network=10.0.0.0",
            "add address=1.1.1.1 interface=loopback network=1.1.1.1",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip route",
            "add dst-address=10.0.0.0/24 gateway=wg0",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=216.239.35.4",
            "add address=216.239.35.8"
        ]
    ]
}
ok: [chr_2] => {
    "full_config.stdout_lines": [
        [
            "# 2024-12-13 14:36:32 by RouterOS 7.16.1",
            "# software id = ",
            "#",
            "/interface bridge",
            "add name=loopback",
            "/interface wireguard",
            "add listen-port=51820 mtu=1420 name=wg0",
            "/routing ospf instance",
            "add disabled=no name=default router-id=2.2.2.2",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.0.0.0/24 endpoint-address=172.20.10.5 endpoint-port=\\",
            "    51820 interface=wg0 name=peer2 public-key=\\",
            "    \"TqqKO9e9OMq34Nk2zESioaVDMLyrHUL+9m3rWM82WWI=\"",
            "/ip address",
            "add address=10.0.0.3/24 interface=wg0 network=10.0.0.0",
            "add address=2.2.2.2 interface=loopback network=2.2.2.2",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip route",
            "add dst-address=10.0.0.1/24 gateway=wg0",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=216.239.35.4",
            "add address=216.239.35.8"
        ]
    ]
}
```
