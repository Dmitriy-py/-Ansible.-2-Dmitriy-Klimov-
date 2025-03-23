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

# Ответ:
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

![Снимок экрана (598)](https://github.com/user-attachments/assets/d41aa3db-edb6-4740-890f-ade3264133d3)

![Снимок экрана (599)](https://github.com/user-attachments/assets/2f29dcf7-766a-483a-9dec-2508d7bba5b7)

![Снимок экрана (600)](https://github.com/user-attachments/assets/6db5eb50-f07f-43a1-8aaf-3367ca054ca1)

![Снимок экрана (601)](https://github.com/user-attachments/assets/7e4b73ec-1f2c-4e20-ac55-049abe7fe0ea)



### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 



### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес.
Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/). Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
4. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
5. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

В качестве решения:
- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.



