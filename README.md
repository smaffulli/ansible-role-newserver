# ansible-role-newserver
An Ansible role to secure all new Ubuntu servers on public networks.
Use it with the sample playbook below, which uses the OpenStack module
to create a new server first and then assigns the newserver role:


    --
      - hosts: localhost
        connection: local
        vars:
          private_key: /home/reed/.ssh/new_id.rsa
        tasks:

        - name: create a Ubuntu server
          os_server:
                cloud: iad2
                name: newserver00
                state: present
                image: Ubuntu-16.04
                flavor_ram: 2048
                key_name: stef
                boot_from_volume: True
                volume_size: 10
                network: public
          register: new_server

        - name: get facts about the server (including its public v4 IP address)
          os_server_facts:
            cloud: iad2
            server: newserver00
            wait: yes
          until: new_server.server.public_v4 != ""
          retries: 5
          delay: 10

        - set_fact: public_v4="{{ new_server.server.public_v4 }}"

        - name: add the server to our ansible inventory
          add_host: hostname={{ public_v4 }} groups=new ansible_ssh_user=dhc-user ansible_ssh_private_key_file={{ private_key }}

        - hosts: new
          gather_facts: no

          tasks:
            - name: Install python2.7
              raw: "sudo apt-get update -qq && sudo apt-get install -qq python2.7 aptitude"

        - hosts: new
          vars:
           ansible_python_interpreter: /usr/bin/python2.7
           logwatch_email: $YOUR_EMAIL@ADDRESS

          become: True

Based on https://gist.github.com/ryane/e0ea8e4a75b140bf799f.
