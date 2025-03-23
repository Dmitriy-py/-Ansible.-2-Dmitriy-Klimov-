# «Дмитрий Климов»

# Домашнее задание к занятию «Ansible.Часть 2»

### Оформление домашнего задания

1. Домашнее задание выполните в [Google Docs](https://docs.google.com/) и отправьте на проверку ссылку на ваш документ в личном кабинете.  
1. В названии файла укажите номер лекции и фамилию студента. Пример названия:  Ansible. Часть 2 — Александр Александров.
1. Перед отправкой проверьте, что доступ для просмотра открыт всем, у кого есть ссылка. Если нужно прикрепить дополнительные ссылки, добавьте их в свой Google Docs.

Вы можете прислать решение в виде ссылки на ваш репозийторий в GitHub, для этого воспользуйтесь [шаблоном для домашнего задания](https://github.com/netology-code/sys-pattern-homework).

---

### Задание 1

**Выполните действия, приложите файлы с плейбуками и вывод выполнения.**

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны: 

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать [официальный сайт](https://kafka.apache.org/downloads) и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

## Ответ:
```
---
- name: Download and extract Apache Kafka
  hosts: all
  become: true
  vars:
    kafka_version: 3.6.1
    kafka_scala_version: 2.13
    kafka_download_url: "https://dlcdn.apache.org/kafka/4.0.0/kafka_2.13-4.0.0.tgz"
    kafka_install_dir: /opt/kafka
    kafka_archive_name: "kafka_{{ kafka_scala_version }}-{{ kafka_version }}.tgz"

  tasks:
    - name: Create installation directory
      file:
        path: "{{ kafka_install_dir }}"
        state: directory
        mode: '0755'

    - name: Download Kafka archive
      get_url:
        url: "{{ kafka_download_url }}"
        dest: /tmp/{{ kafka_archive_name }}
        checksum: "sha512:00722ab0a6b954e0006994b8d589dcd8f26e1827c47f70b6e820fb45aa35945c19163b0f188caf0caf976c11f7ab005fd368c54e5851e899d2de687a804a5eb9"  # Added checksum for>

    - name: Extract Kafka archive
      unarchive:
        src: /tmp/{{ kafka_archive_name }}
        dest: "{{ kafka_install_dir }}"
        remote_src: yes
        creates: "{{ kafka_install_dir }}/kafka_{{ kafka_scala_version }}-{{ kafka_version }}/LICENSE"  # Check if extraction already happened
        owner: root
        group: root

    - name: Clean up downloaded archive
      file:
        path: /tmp/{{ kafka_archive_name }}
        state: absent
```
![Снимок экрана (597)](https://github.com/user-attachments/assets/ef74af3d-0724-4540-9612-8e720e93c09e)

```
---
- name: Install and configure tuned
  hosts: all
  become: true

  tasks:
    - name: Install tuned package
      package:
        name: tuned
        state: present

    - name: Start and enable tuned service
      systemd:
        name: tuned
        state: started
        enabled: yes
```
![Снимок экрана (598)](https://github.com/user-attachments/assets/d41aa3db-edb6-4740-890f-ade3264133d3)

![Снимок экрана (599)](https://github.com/user-attachments/assets/2f29dcf7-766a-483a-9dec-2508d7bba5b7)

```
---
- name: Change MOTD
  hosts: all
  become: true
  vars:
    motd_message: |
      #########################################
      #                                       #
      #        Welcome to the System!        #
      #                                       #
      #########################################
      This system is managed by Ansible.
      Please be careful!

  tasks:
    - name: Set MOTD content
      copy:
        dest: /etc/motd
        content: "{{ motd_message }}"
        owner: root
        group: root
        mode: '0644'
```

![Снимок экрана (600)](https://github.com/user-attachments/assets/6db5eb50-f07f-43a1-8aaf-3367ca054ca1)

![Снимок экрана (601)](https://github.com/user-attachments/assets/7e4b73ec-1f2c-4e20-ac55-049abe7fe0ea)



### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 

## Ответ:

```
--
- name: Change MOTD with dynamic information
  hosts: all
  become: true

  tasks:
    - name: Get IP address
      setup:
        filter: ansible_default_ipv4
      delegate_to: localhost  # Get IP of the managed host, not the controller
      run_once: true
      register: ip_address_info

    - name: Get hostname
      shell: hostname
      register: hostname_output
      delegate_to: localhost
      run_once: true

    - name: Set MOTD content
      copy:
        dest: /etc/motd
        content: |
          #########################################
          #                                       #
          #        Welcome to the System!        #
          #                                       #
          #########################################
          Hostname: {{ hostname_output.stdout }}
          IP Address: {{ ip_address_info.ansible_facts.ansible_default_ipv4.address }}
          Have a nice day, System Administrator!
          This system is managed by Ansible.
          Please be careful!
        owner: root
        group: root
        mode: '0644'
```

![Снимок экрана (602)](https://github.com/user-attachments/assets/e3515929-454d-46d0-a221-7b53b2726e9c)


### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес.
Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/). Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
4. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
5. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

## Ответ:

### cat apache_system_info/tasks/main.yml
```
---
# tasks file for apache_system_info
- name: Install Apache
  package:
    name: "{{ apache_package_name }}"
    state: present

- name: Open port {{ http_port }} in firewall (Firewalld) - using firewalld
  firewalld:
    port: "{{ http_port }}/tcp"
    permanent: yes
    state: enabled
  when: ansible_os_family == "RedHat"

- name: Open port {{ http_port }} in firewall (UFW) - using ufw
  ufw:
    rule: allow
    port: "{{ http_port }}"
    proto: tcp
  when: ansible_os_family == "Debian" or ansible_os_family == "Ubuntu"

- name: Configure index.html with system info
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
  notify: restart apache

- name: Start and enable Apache service
  service:
    name: "{{ apache_service_name }}"
    state: started
    enabled: true

- name: Check website availability
  uri:
    url: "http://localhost"
    status_code: 200
```

### cat apache_playbook.yml

```
---
- name: Deploy Apache with system information
  hosts: all
  become: true

  roles:
    - apache_system_info
  tasks:
    - name: Debug CPU Model
      debug:
        var: ansible_processor

    - name: Debug Total RAM
      debug:
        var: ansible_memtotal_mb

    - name: Debug Disk Info
      debug:
        var: ansible_devices
```

### cat apache_system_info/templates/index.html.j2

```
<!DOCTYPE html>
<html>
<head>
  <title>System Information</title>
</head>
<body>
  <h1>System Information</h1>
  <ul>
    <li><strong>Hostname:</strong> {{ ansible_hostname }}</li>
    <li><strong>IP Address:</strong> {{ ansible_default_ipv4.address }}</li>
    <li><strong>CPU Model:</strong> {{ ansible_processor }}</li>
    <li><strong>Total RAM:</strong> {{ ansible_memtotal_mb }} MB</li>
    <li><strong>First HDD Size:</strong> {{ ansible_devices.sda.size }} </li>
  </ul>
</body>
</html>

```

![Снимок экрана (608)](https://github.com/user-attachments/assets/fc3b48e4-11ae-4f3b-a418-a5ff8ea8e33a)


![Снимок экрана (609)](https://github.com/user-attachments/assets/f5f4397e-6752-432e-8fe3-d51f257b58ec)


![Снимок экрана (610)](https://github.com/user-attachments/assets/1c25c7a9-73d3-425c-835b-8bfda9cc27fa)


![Снимок экрана (618)](https://github.com/user-attachments/assets/4a578cb9-df86-4e39-87ed-a01245f5f034)




