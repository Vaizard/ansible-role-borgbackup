Borgbackup
==========

Ansible [Borgbackup](https://borgbackup.readthedocs.io/en/stable/) role.

Features:
 * Repository or binary installation
 * Schedules regular backup jobs
 * Schedules regular prune jobs to keep backup windows clean
 * Flexible configuration to list backup targets

Role Variables
--------------

see `defaults/main.yml`

Example Playbook
----------------

    - hosts: all
      roles:
        - role: mage-borgbackup
          borgbackup_client: true
          borgbackup_client_backup_server: backup01.example.com
          borgbackup_client_jobs:
            - name: system
              day: "*"
              hour: "0"
              minute: "{{ 59 | random }}"
              directories:
                - /etc
                - /home
                - /var
              excludes:
                - 're:^/var/lib/apt'
                - 're:^/var/[^/]+\/cache/'
          borgbackup_prune_jobs:
            - name: system
              prune_options: "--keep-daily=7 --keep-weekly=4"
              day: "*"
              hour: "8"
              minute: "0"
    - hosts: backup01.example.com
      roles:
        - role: mage-borgbackup
          borgbackup_server: true
        - role: 
          
You can also easily assign client and server attributes from your inventory with something similar to the following:

    borgbackup_client: "{{ (inventory_hostname in groups.borgbackup_server)|ternary(False, True) }}"
    borgbackup_client_backup_server: "{{ groups.borgbackup_server[0] }}"


Limitations
-----------

Due to how this role was designed, it will work **ONLY FOR BACKUP SERVERS WITH SSH LISTENING ON PORT 22**. The reasons are:

- `delegate_to: "{{ borgbackup_client_backup_server }}"` will not accept a host:port combination, you will need to define
   a named connection in the /ansible/hosts file and use that instead
- Various commands will have to be rewritten, i.e. from
  `command: "{{ borgbackup_binary }} list {{ borgbackup_server_user }}@{{ borgbackup_client_backup_server }}:{{ item.name }}"`
  to
  `command: "{{ borgbackup_binary }} list 'ssh://{{ borgbackup_server_user }}@{{ borgbackup_client_backup_server }}:{{ item.name }}'"`
- Addtionally `Repository path not allowed` errors will pop up and force you to define full repo path as required by the 
  different syntax when using quotation and the ssh:// prefix.

Tried this, but the role felt too hacky. I decided not to commit these changes and document this here until I come with a 
better idea.


License
-------

Apache 2.0



