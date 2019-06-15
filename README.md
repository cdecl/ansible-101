## ansible-101

### Install
- centos 
```bash
sudo yum install ansible
```

- pip 으로 설치 (user)
```bash
pip install --user ansible
```

### 용어 

- Inventory : 관리하는 원격 서버 목록 
	- default path : /etc/ansible/hosts
- Module : Task 를 실행하는 방법 (모듈) 
	- https://docs.ansible.com/ansible/latest/modules/modules_by_category.html
	- e.g command, shell, copy, service 
- Playbook : Task 실행 정의서 (yaml)


### Config
- path : /etc/ansible/ansible.cfg
- ignore ansible ssh authenticity
	- https://stackoverflow.com/questions/32297456/how-to-ignore-ansible-ssh-authenticity-checking

```bash
[defaults]
host_key_checking = False
```

- 참고 : How to disable strict host key checking in ssh?
	- https://askubuntu.com/questions/87449/how-to-disable-strict-host-key-checking-in-ssh


### Inventory
ssh-keygen 을 통해 키를 생성하고 public 키를 관리되는 서버에 등록 후 사용  
```
ssh-keygen

ssh-copy-id [server-ip] 
```
- Inventory 서버 등록 
	- /etc/ansible/hosts 
```

[webservers]
192.168.1.100
192.168.1.110
192.168.1.111
```

- Inventory 지정 
	- all : 지정된 모든 서버 
	- webservers : webservers 라고 정의된 서버 목록
	- 192.168.1.100 : 192.168.1.100 서버 


### Ad-hook Execute
- 주요 인수 

```
Usage: ansible <host-pattern> [options]
  -a MODULE_ARGS, --args=MODULE_ARGS                                                                       
                        module arguments                                                                   
  -f FORKS, --forks=FORKS                                                                                  
                        specify number of parallel processes to use                                        
                        (default=5)                                                                        
  -h, --help            show this help message and exit                                                    
  -i INVENTORY, --inventory=INVENTORY, --inventory-file=INVENTORY                                          
                        specify inventory host path or comma separated host                                
                        list. --inventory-file is deprecated                                               
  -m MODULE_NAME, --module-name=MODULE_NAME                                                                
                        module name to execute (default=command)                                           
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable                                       
                        connection debugging)                                                              
  --version             show program's version number, config file location,                               
                        configured module search path, module location,                                    
                        executable location and exit                                                       
    -b, --become        run operations with become (does not imply password                                
                        prompting)                                                                         
```

#### - Example 
- hostname 확인
```
# webservers 목록 실행 
ansible webservers -m command -a "hostname" 

# default module (생략가능) : -m command   
ansible webservers -a "hostname" 

# 모든 서버
ansible all -a "hostname" 
```

- Ping : https://docs.ansible.com/ansible/latest/modules/ping_module.html
```
ansible webservers -m ping
```

- Copy : https://docs.ansible.com/ansible/latest/modules/copy_module.html
```
ansible webservers -m copy -a "src=file.txt dest=/home/cdecl/web/"
```

### Playbook 


#### - Example
- Playbook 실행 
```bash
# playbook 실행
ansible-playbook ping.yml
```

- Shell 모듈 
```yaml
- name: get hostname
  hosts: webservers
  tasks:
    - shell: "hostnaem"
      register: out
    - debug: "var=stdout_lines" 
```

- Ping 모듈 
```yaml
- name: ping test 
  hosts: webservers
  tasks:
    - ping:
```

- Service 모듈
```yaml
- name: nginx service start
  hosts: webservers
  tasks:
    - service: "name=nginx state=started"
      become: yes
```
```yaml
- name: nginx service start
  hosts: webservers
  tasks:
    - service:
        name: nginx
        state: started
      become: yes
```