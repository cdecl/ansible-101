## Ansible-101

### Ansible Install
- Centos 7 
	- 기본적으로 Python 2.7 기반으로 설치 됨 
	- 추후 pywinrm 패키지가 필요할때, python2-pip 패키지 추가 설치 필요 
```bash
sudo yum install ansible
```

- 대안으로 pip 으로 설치 하는 방법 (user)
```bash
pip install ansible # --user 
```

### 용어 정의 

- Inventory : 관리하는 원격 서버 목록 
	- 지정하지 않으면 Default Path 의 hosts 파일을 참조 
		- /etc/ansible/hosts
- Module : Task 를 실행하는 방법 (모듈) 
	- https://docs.ansible.com/ansible/latest/modules/modules_by_category.html
	- e.g. - command, shell, copy, service 
- Playbook : Task 실행 프로세스 정의서 
	- Yaml 파일로 작성 
- 멱등성(idempotence) : 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질을 의미한다.


### Ansbile Config 
- Path : /etc/ansible/ansible.cfg
- Ignore ansible ssh authenticity : ssh 최초 접속시 인증을 host key 체크 생략  
	- https://stackoverflow.com/questions/32297456/how-to-ignore-ansible-ssh-authenticity-checking

```bash
[defaults]
host_key_checking = False
```

- Callback plugins 
	- minimal stdout 
```
[defaults]
stdout_callback = minimal
```

- 참고 : How to disable strict host key checking in ssh?
	- https://askubuntu.com/questions/87449/how-to-disable-strict-host-key-checking-in-ssh


### Inventory 구성 
- Public key 등록 사용 방법 : 키를 생성하고 public 키를 관리되는 서버에 등록 후 사용  

```
# Key 생성 
ssh-keygen

# Public key 등록
ssh-copy-id [server-ip] 
```

- Inventory 서버 등록 : /etc/ansible/hosts 
	- 기본디렉토리의 Host 파일을 사용하지 않는다면 -i 옵션으로 Inventory 파일 지정 
	- https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#list-of-behavioral-inventory-parameters
```
# ansible host 파일 예제

192.168.1.5

[db]
192.168.1.10

[webservers]
192.168.1.100
192.168.1.110

# hosts 파일에 등록된 경우 
[apiserver]
apiserver1
apiserver2

# host명으로 등록
server1		ansible_host=192.168.1.50
server2		ansible_host=192.168.1.51

```

- Host 지정 방법 
	- `all` : 해당 Inventory의 모든 서버 
	- `webservers` : webservers 라고 정의된 서버 목록
	- `192.168.1.100` : 192.168.1.100 서버 


### Ad-hook Execute
- ansible 명령을 통한 inline 실행 
- 주요 인수 

```
Usage: ansible <host-pattern> [options]
  -a MODULE_ARGS, --args=MODULE_ARGS   module arguments                                                                   
  -e EXTRA_VARS, --extra-vars=EXTRA_VARS
    set additional variables as key=value or YAML/JSON, if filename prepend with @  
  -f FORKS, --forks=FORKS 
  	specify number of parallel processes to use (default=5)                                                                        
  -h, --help            show this help message and exit                                                     
  -i INVENTORY, --inventory=INVENTORY, --inventory-file=INVENTORY                                           
  	specify inventory host path or comma separated host                                
  	list. --inventory-file is deprecated                                               
  -m MODULE_NAME, --module-name=MODULE_NAME                                                                 
  	module name to execute (default=command)                                           
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable connection debugging)                                                              
  --version             show program's version number, config file location,                                
  	configured module search path, module location,                                    
  	executable location and exit                                                       
  -b, --become        run operations with become (does not imply password prompting)                                                                         
```

#### - Example 
- hostname 확인
```
# webservers 목록 실행 
# command 모듈, "hostname" 인수
ansible webservers -m command -a "hostname" 

# default module (생략가능) : -m command   
ansible webservers -a "hostname" 

# 모든 서버
ansible all -a "hostname" 
```

- 기타 모듈 사용 
```
# ping 모듈 
ansible webservers -m ping

# copy 모듈 : local → host 
ansible webservers -m copy -a "src=file.txt dest=/home/cdecl/web/"

# service 모듈 : kubelet 서비스를 시작 
ansible webservers -m service -a "name=kubelet state=started"
```


### Playbook 사용 
일련의 Task(작업)를 기술하여, 프로세스를 실행하는 방법 

- 간략한 설명 
	- name : 작업을 설명하는 식별자 (주석, 생략가능)
		- 모듈 내에서의 name 은 다른 의미 
	- gather_facts : (yes | no ) Gathering Facts 작업 실행 및 생략  
		- Facts : 작업 시작전 Host 의 Name, CPU, Memory 등을 수집하는 Setup 모듈
	- hosts : Inventory 내의 hosts 지정 
	- tasks : 작업 목록들 (배열)
	- register : 작업을 해당 이름으로 등록, debug 항목에서 표준 출력을 받아오기 위해 사용 
	- debug : debug 용 작업들 (msg, stdout 출력)
	- become : root 권한으로 실행 
	- become_user : sudo 를 통해 실행할 계정 
	- 변수값 평가 : " {{ 변수 }} "
		- -e 파라미터를 통해 전달 가능

#### - Example

- Ping 모듈 테스트

```yaml
- name: ping test 
  hosts: webservers
  gather_facts: no
  tasks:
    - ping:
```

```bash
ansible-playbook ping.yml
```

- Shell 모듈 : out 이란 이름으로 작업을 등록 
```yaml
- name: get hostname
  hosts: localhost
  tasks:
    - shell: "hostname"
	  register: out  # out 이란 이름으로 작업을 등록 
	  
	- debug: "var=out.stdout_lines"  # out 의 stdout 을 모두 출력 
```

```bash
> ansible-playbook hostname.yml

localhost | CHANGED | rc=0 >>
centos24

localhost | SUCCESS => {
    "out.stdout_lines": [
        "centos24"
    ]
}
```

- Service 모듈
```yaml
- name: nginx service start
  hosts: localhost
  # gather_facts: no
  tasks:
    - service: # nginx 서비스를 시작
        name: nginx
        state: started
#       - service: "name=nginx state=started"  #  inline 으로 표현 가능
      become: yes  # root 권한 실행
```

```bash
> # changed=1 : 상태 변화 있음 
> ansible-playbook nginx.yml

PLAY [nginx service start] 
TASK [service] ************changed: [localhost]
PLAY RECAP ****************localhost                  : ok=1    changed=1    unreachable=0    failed=0


> # changed=0 : 상태 변화 없음 → 이미 started 상태 
> ansible-playbook nginx.yml

PLAY [nginx service start] 
TASK [service] ************changed: [localhost]
PLAY RECAP ****************localhost                  : ok=1    changed=0    unreachable=0    failed=0

```

- Kubenetes init 예제
	- https://github.com/cdecl/kubernetes-101/tree/master/ansible

```yml
- hosts: kubem
  tasks:
    - name: initialize the cluster
      become: yes
      shell: kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors all >> cluster_initialized.txt
      args:
        chdir: $HOME
        creates: cluster_initialized.txt  # 멱등성확보, 파일이 있으면 해당 작업 스킵 

    - name: create .kube directory
      file:
        path: $HOME/.kube
        state: directory
        mode: 0755

    - name: copy admin.conf to user's kube config
      become: yes
      copy:
        src: /etc/kubernetes/admin.conf
        dest: /home/cdecl/.kube/config
        remote_src: yes

    - name: install Pod network
      shell: kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml >> pod_network_setup.txt
      args:
        chdir: $HOME
        creates: pod_network_setup.txt
```		


#### - Windows Host 사용 
- Windows Host로 사용하는 방법
	- ssh
	- winrm : https://github.com/cdecl/winrm-101 

```powershell
# host 설정 : Service Enable 
winrm qc 

# 인증 및 AllowUnencrypted 설정 
winrm set winrm/config/service/Auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'

```

```bash
# winrm 을 사용하기 위한 python 모듈 설치 
pip install pywinrm   # --user 
```

- Inventory 구성 

```ini

[win]
192.168.28.25

[win:vars]
ansible_user=rundeck
ansible_password=<passowrd>
ansible_connection=winrm
ansible_port=5985  
# ansible_winrm_server_cert_validation=ignore
```

- Windows 모듈 
	- https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html
	- win_command – Executes a command on a remote Windows node
	- win_copy – Copies files to remote locations on windows hosts
	- win_shell – Execute shell commands on target hosts (powershell)
	- win_chocolatey – Manage packages using chocolatey


```yaml
- name: ping test 
  hosts: win
  gather_facts: no
  tasks:
	- win_ping:
	
- name: Install git from a pre configured source (win_chocolatey_source)
  hosts: win
  tasks:
    - win_chocolatey:
        name: git
```


