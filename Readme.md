1. К существующему playbook мы добавили установку kibana.   
2. Для выполнения playbook я модифицировал файл prod.yml, где указываются хосты. В моем случае я разворачивал установку в docker контейнере.
```
---
elasticsearch:
  hosts:
    ubuntu:
      ansible_connection: docker

kibana:
  hosts:
    ubuntu:
      ansible_connection: docker
```
3. - name: Install Kibana  
  hosts: kibana **- настройка хостов**  
  tasks:  
    - name: Upload tar.gz Kibana from remote URL - **загрузка с внешней ссылки дистрибутива**  
      get_url:  
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"  
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz" - **Назначение для загрузки**  
        mode: 0755  
        timeout: 60 - **Таймаут**  
        force: true - **Принудительная загрузка несмотря на наличие дистрибутива в директории**   
        validate_certs: false - **Игнорирование сертификатов при подключении**   
      register: get_kibana - **Переменная для контроля выполнения**     
      until: get_kibana is succeeded    
      tags: kibana - **ТЕГ kibana**    
    - name: Create directrory for Kibana - **Создание директории для загрузки на хост**   
      file:   
        state: directory   
        path: "{{ kibana_home }}" - **Директория указанная в vars.yml**   
      tags: kibana   
    - name: Extract Kibana in the installation directory    
      become: true - **Получение привелегий для удаленной установки**   
      unarchive: - **Разархивирование архива на удаленной машине**   
        copy: false    
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"    
        dest: "{{ kibana_home }}"    
        extra_opts: [--strip-components=1]    
        creates: "{{ kibana_home }}/bin/kibana" - **директория разархивирования**   
      tags:   
        - kibana   
    - name: Set environment kibana   
      become: true   
      template:   
        src: templates/kib.sh.j2 - **Проброс тэплейт файла на удаленную машину**   
        dest: /etc/profile.d/kib.sh - **Сравнение файла на удаленной машине и пробрасываемого файла. В случае отличий, его перезапись на пробрасываемый**    
      tags: kibana    
