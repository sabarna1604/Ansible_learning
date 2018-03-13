Ansible on Windows :


	Referred Link :

		https://medium.com/the-sysadmin/managing-windows-machines-with-ansible-60395445069f


 Steps :

	In Windows Machine :

	STEP 1 - start WinRM service in command prompt (run command prompt as administrator) using 
	    
		@powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((new-object net.webclient).DownloadString('http://54.235.238.79/ConfigureRemotingForAnsible.ps1'))"

	ERROR MESSAGE :

	 if you run this command sometimes you will get an error telling that "change your network connection to either private or Domain "

	SOLUTION to OVERCOME ERROR :
	
		open powershell (run as administrator) and copy the following commands :
			
			1) $networkListManager = [Activator]::CreateInstance([Type]::GetTypeFromCLSID([Guid]"{DCB00C01-570F-4A9B-8D69-199FDBA5723B}")) 

			2) $connections = $networkListManager.GetNetworkConnections()

			3) $connections | % {$_.GetNetwork().SetCategory(1)}


	after running these commands if you run cmd command again, it will run without error.

	STEP 2 Install chocolately either using powershell or command prompt :
     
              ## Install chocolately via powershell using 

		Set-ExecutionPolicy Bypass -Scope Process -Force; iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex

	       ## if you want to install via cmd use ,

		@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

         Now chocolately will be installed.


	IN LOCAL UBUNTU MACHINE :

		STEP 1 - create a folder group_vars and inside group_vars create a yml file named windows (give path where you want to create these folder). username and password will be your system login name and password.

	CONTENT :

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat group_vars/windows.yml 

ansible_user: Admatic-51
ansible_password: redhat!@#
ansible_port: 5986
ansible_connection: winrm
# The following is necessary for Python 2.7.9+ when using default WinRM self-signed certificates:
ansible_winrm_server_cert_validation: ignore

		STEP 2 - Give hosts name of windows machine .

	CONTENT :

	     admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat hosts

[windows]
192.168.0.102
	 

	       STEP 3 - Now write your playbook, module will be different for windows. In linux, we use "ping" to ping a hosts but in windows we use "win_ping"

	PING MODULE :

	PLAYBOOK CONTENT:

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat ping_windows.yml 
---
- name: test chocolatey with ansible
  hosts: windows
  tasks:
    - name: Ping
      win_ping:

	OUTPUT :

	 admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook ping_windows.yml -i hosts 

PLAY [test chocolatey with ansible] ********************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.0.102]

TASK [Ping] ********************************************************************
ok: [192.168.0.102]

PLAY RECAP *********************************************************************
192.168.0.102              : ok=2    changed=0    unreachable=0    failed=0   

	
	COPY MODULE :

	PLAYBOOK CONTENT :

	admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat copy_windows.yml 
---
- name: test chocolatey with ansible
  hosts: windows
  tasks:
    - name: Ping
      win_copy:
        src: /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module
        dest: C:\Users\Interview\Documents
	

	OUTPUT :

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook copy_windows.yml -i hosts 

PLAY [test chocolatey with ansible] ********************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.0.102]


TASK [Ping] ********************************************************************
changed: [192.168.0.102] => {
    "changed": true, 
    "dest": "C:\\Users\\Interview\\Documents", 
    "operation": "folder_copy", 
    "src": "/home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module"
}


PLAY RECAP *********************************************************************
192.168.0.102              : ok=2    changed=1    unreachable=0    failed=0   


	INSTALL VLC USING WIN_CHOCOLATELY :

	PLAYBOOK CONTENT :

	admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat install_vlc.yml 
---
- name: test chocolatey with ansible
  hosts: windows
  tasks:
    - name: Install vlc
      win_chocolatey:
        name: vlc 
        state: present

	OUTPUT :

	admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook install_vlc.yml -i hosts -vvvv

PLAY [test chocolatey with ansible] ********************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.0.102]


TASK [Install vlc] *************************************************************
ok: [192.168.0.102] => {
    "changed": false
}


PLAY RECAP *********************************************************************
192.168.0.102              : ok=2    changed=0    unreachable=0    failed=0   


	COPY MODULE (WINDOWS TO WINDOWS) :

	PLAYBOOK CONTENT :

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat copy_windows_to_windows.yml 
---
- name: test chocolatey with ansible
  hosts: windows
  tasks:
    - name: copy
      win_copy:
        src: C:\Users\Interview\Documents\Windows-playbook-module
        dest: C:\Users\Admatic\Documents
        remote_src: True


	OUTPUT :

		
admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook copy_windows_to_windows.yml -i hosts 

PLAY [test chocolatey with ansible] ********************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.0.114]

TASK [copy] ********************************************************************
ok: [192.168.0.114]

PLAY RECAP *********************************************************************
192.168.0.114              : ok=2    changed=0    unreachable=0    failed=0   


	COPY MODULE (WINDOWS TO UBUNTU) :

	PLAYBOOK CONTENT :

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat copy_module_windows_ubuntu.yml 
---
- name: test copy module with ansible
  hosts: windows
  tasks:
    - name: copy
      win_copy:
        src: C:\Users\Admatic\Downloads\dataset\testing\0
        dest: /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module
        remote_src: True

	OUTPUT :

	admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook copy_module_windows_ubuntu.yml -i hosts  

PLAY [test copy module with ansible] *******************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.0.114]

TASK [copy] ********************************************************************
ok: [192.168.0.114]

PLAY RECAP *********************************************************************
192.168.0.114              : ok=2    changed=0    unreachable=0    failed=0   



	ADD_HOSTS MODULE :

	PLAYBOOK CONTENT :

	admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat addhosts_module.yml 
---
- name: test ping with ansible
  hosts: windows
  tasks:
      - name : add hosts
        add_host:
         hostname: 192.168.0.105
         ansible_user: Interview
         ansible_password: admatic123
         ansible_port: 5986
         ansible_connection: winrm
         # The following is necessary for Python 2.7.9+ when using default WinRM self-signed       certificates:
         ansible_winrm_server_cert_validation: ignore

      - name: Ping
        win_ping:


	OUTPUT :

	admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook addhosts_module.yml -i hosts -vvvv  
ansible-playbook 2.4.3.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/admatic/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
Using /etc/ansible/ansible.cfg as config file
setting up inventory plugins
Parsed /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module/hosts inventory source with ini plugin
Loading callback plugin default of type stdout, v2.0 from /usr/lib/python2.7/dist-packages/ansible/plugins/callback/default.pyc

PLAYBOOK: addhosts_module.yml **************************************************
1 plays in addhosts_module.yml

PLAY [test ping with ansible] **************************************************

TASK [Gathering Facts] *********************************************************
Using module file /usr/lib/python2.7/dist-packages/ansible/modules/windows/setup.ps1
<192.168.0.114> ESTABLISH WINRM CONNECTION FOR USER: Admatic on PORT 5986 TO 192.168.0.114
checking if winrm_host 192.168.0.114 is an IPv6 address
EXEC (via pipeline wrapper)
ok: [192.168.0.114]
META: ran handlers

TASK [add hosts] ***************************************************************
task path: /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module/addhosts_module.yml:5
creating host via 'add_host': hostname=192.168.0.105
changed: [192.168.0.114] => {
    "add_host": {
        "groups": [], 
        "host_name": "192.168.0.105", 
        "host_vars": {
            "ansible_connection": "winrm", 
            "ansible_password": "admatic123", 
            "ansible_port": 5986, 
            "ansible_user": "Interview", 
            "ansible_winrm_server_cert_validation": "ignore"
        }
    }, 
    "changed": true
}

TASK [Ping] ********************************************************************
task path: /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module/addhosts_module.yml:15
Using module file /usr/lib/python2.7/dist-packages/ansible/modules/windows/win_ping.ps1
<192.168.0.114> ESTABLISH WINRM CONNECTION FOR USER: Admatic on PORT 5986 TO 192.168.0.114
checking if winrm_host 192.168.0.114 is an IPv6 address
EXEC (via pipeline wrapper)
ok: [192.168.0.114] => {
    "changed": false, 
    "ping": "pong"
}
META: ran handlers
META: ran handlers

PLAY RECAP *********************************************************************
192.168.0.114              : ok=3    changed=1    unreachable=0    failed=0   


	FETCH MODULE :

	PLAYBOOK CONTENT :

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ cat fetch_module.yml 
---
- name: test fetch with ansible
  hosts: windows
  tasks:
      - name : fetch module
        fetch:
         src: C:\Users\Interview\Documents\Windows-playbook-module\hosts
         dest: /home/admatic/ansible-book/ansible


	OUTPUT :

		admatic@admatic-Aspire-E5-573G:~/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module$ ansible-playbook fetch_module.yml -i hosts -vvvv  
ansible-playbook 2.4.3.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/admatic/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
Using /etc/ansible/ansible.cfg as config file
setting up inventory plugins
Parsed /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module/hosts inventory source with ini plugin
Loading callback plugin default of type stdout, v2.0 from /usr/lib/python2.7/dist-packages/ansible/plugins/callback/default.pyc

PLAYBOOK: fetch_module.yml *****************************************************
1 plays in fetch_module.yml

PLAY [test fetch with ansible] *************************************************

TASK [Gathering Facts] *********************************************************
Using module file /usr/lib/python2.7/dist-packages/ansible/modules/windows/setup.ps1
<192.168.0.114> ESTABLISH WINRM CONNECTION FOR USER: Admatic on PORT 5986 TO 192.168.0.114
checking if winrm_host 192.168.0.114 is an IPv6 address
EXEC (via pipeline wrapper)
ok: [192.168.0.114]
META: ran handlers

TASK [fetch module] ************************************************************
task path: /home/admatic/ansible-book/ansible/Sabarna_playbook/Windows-playbook-module/fetch_module.yml:5
Using module file /usr/lib/python2.7/dist-packages/ansible/modules/windows/win_stat.ps1
<192.168.0.114> ESTABLISH WINRM CONNECTION FOR USER: Admatic on PORT 5986 TO 192.168.0.114
checking if winrm_host 192.168.0.114 is an IPv6 address
EXEC (via pipeline wrapper)
<192.168.0.114> FETCH "C:\Users\Interview\Documents\Windows-playbook-module\hosts" TO "/home/admatic/ansible-book/ansible/192.168.0.114/C:/Users/Interview/Documents/Windows-playbook-module/hosts"
changed: [192.168.0.114] => {
    "changed": true, 
    "checksum": "11a6b44c7dee26bcefbc303c7de130f26b8d596c", 
    "dest": "/home/admatic/ansible-book/ansible/192.168.0.114/C:/Users/Interview/Documents/Windows-playbook-module/hosts", 
    "md5sum": "fcc0073ecc0b5b2c650579da7f8bf2f0", 
    "remote_checksum": "11a6b44c7dee26bcefbc303c7de130f26b8d596c", 
    "remote_md5sum": null
}
META: ran handlers
META: ran handlers

PLAY RECAP *********************************************************************
192.168.0.114              : ok=2    changed=1    unreachable=0    failed=0   






	



		
