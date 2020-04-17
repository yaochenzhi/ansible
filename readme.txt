[config]

        ANSIBLE_CONFIG (environment variable if set)
        ansible.cfg (in the current directory)
        ~/.ansible.cfg (in the home directory)
        /etc/ansible/ansible.cfg

        ansible-config

[inventory]

        https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#id12

        --
        mail.example.com

        [webservers]
        foo.example.com
        bar.example.com

        [dbservers]
        one.example.com
        two.example.com
        three.example.com http_port=80 maxRequestsPerChild=808
        jumper ansible_port=5555 ansible_host=192.0.2.50

        [dbservers:vars]
        ntp_server=ntp.atlanta.example.com
        proxy=proxy.atlanta.example.com

        --
        all:
          hosts:
            mail.example.com:
          children:
            webservers:
              hosts:
                foo.example.com:
                bar.example.com:
            dbservers:
              hosts:
                one.example.com:
                two.example.com:
                three.example.com:
                  http_port: 80
                  maxRequestsPerChild: 808
                jumper:
                  ansible_port: 5555
                  ansible_host: 192.0.2.50
              vars:
                ntp_server: ntp.atlanta.example.com
                proxy: proxy.atlanta.example.com

        --
        all:
          hosts:
            mail.example.com:
          children:
            webservers:
              hosts:
                foo.example.com:
                bar.example.com:
            dbservers:
              hosts:
                one.example.com:
                two.example.com:
                three.example.com:
            east:
              hosts:
                foo.example.com:
                one.example.com:
                two.example.com:
            west:
              hosts:
                bar.example.com:
                three.example.com:
            prod:
              children:
                east:
            test:
              children:
                west:

        Adding ranges of hosts

        www[01:50].example.com



        [atlanta]
        host1
        host2

        [atlanta:vars]
        ntp_server=ntp.atlanta.example.com
        proxy=proxy.atlanta.example.com


        atlanta:
          hosts:
            host1:
            host2:
          vars:
            ntp_server: ntp.atlanta.example.com
            proxy: proxy.atlanta.example.com

        ** By default variables are merged/flattened to the specific host before a play is run.

        /etc/ansible/group_vars/dbservers
        --
        ntp_server:ntp.atlanta.example.com
        proxy:proxy.atlanta.example.com

        /etc/ansible/group_vars/dbservers/ntp/server
        --
        ntp_server:ntp.atlanta.example.com

        /etc/ansible/group_vars/dbservers/proxy
        proxy:proxy.atlanta.example.com



        ** Using multiple inventory sources
            ansible-playbook get_logs.yml -i staging -i production

[ patterns ]

    https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html#intro-patterns
        
        Common patterns
        
            Description	Pattern(s)	Targets
            All hosts	            all (or *)	 
            One host	            host1	 
            Multiple hosts	        host1:host2 (or host1,host2)	 
            One group	            webservers	 
            Multiple groups	        webservers:dbservers	all hosts in webservers plus all hosts in dbservers
            Excluding groups	    webservers:!atlanta	all hosts in webservers except those in atlanta
            Intersection of groups	webservers:&staging	any hosts in webservers that are also in staging

        Using variables in patterns
    
            webservers:!{{ excluded }}:&{{ required }}

        Using group position in patterns
            
            webservers[0]
            webservers[-1]
            webservers[0:2]
            webservers[:2]
            webservers[2:]

        Using regexes in patterns
        
            ~(foo|bar).*\.example\.com

        Patterns and ansible-playbook flags
    
    
[ adhoc ]

    https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html#intro-adhoc
    
    $ ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]


[module]

    https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html
    
    command:
        # https://docs.ansible.com/ansible/latest/modules/command_module.html#command-module
        # cmd(adhot default)
            $ ansible all -a "echo content | grep content"
    shell:
        # https://docs.ansible.com/ansible/latest/modules/shell_module.html#shell-module
        # cmd through shell(/bin/shell). so variables like $HOME and operations like "<", ">", "|", ";" and "&" will work
            $ ansible all -m shell -a "echo content | grep content"
    copy:
        ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"
    file:
        # delete
            $ ansible webservers -m file -a "dest=/path/to/c state=absent"
        # chmod chown
            $ ansible webservers -m file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
        # mkdir
            $ ansible webservers -m file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
    yum:
        # install if not 
            $ ansible webservers -m yum -a "name=acme state=present"
        # install a specific version of a package if not 
            $ ansible webservers -m yum -a "name=acme-1.5 state=present"
        # install latest if not             
            $ ansible webservers -m yum -a "name=acme state=latest"
        # uninstall if not 
            $ ansible webservers -m yum -a "name=acme state=absent"
    user：
        # 
            $ ansible all -m user -a "name=foo password=<crypted password here>"
        # 
            $ ansible all -m user -a "name=foo state=absent"
    # To put it simply, we are just ensuring the target to be the target status
    
    service:
        $ ansible webservers -m service -a "name=httpd state=started"
        $ ansible webservers -m service -a "name=httpd state=restarted"
        $ ansible webservers -m service -a "name=httpd state=stopped"
    
    setup:
        # https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html#gathering-facts
        # https://docs.ansible.com/ansible/latest/modules/setup_module.html#setup-module
        
        
        # filter ansible facts when using default gather_subset
            ansible test -m setup -a 'filter=*user*'
        # gather_subset specification
            ansible test -m setup -a 'gather_subset=network'
