# NGINX App Protect Ansible Playbook
## Example
Only require to define configuration object in nap_var.yml. Currently only build app protect policy based on those variable. Can be easily extended to includes other variable.

Example

$ ansible-playbook -i inventory nap_play.yml

PLAY [### PLAY 01 ### - Create NGINX App Protect Policy] *************************************************************

TASK [# TASK 01 # - Generating NGINX App Protect policy enforcement] *************************************************
ok: [localhost]

PLAY [### PLAY 02 ### - Transfering App Protect Policy Enforcement] **************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [192.168.201.16]

TASK [# TASK 01 # - Transferring policy for enforcement] *************************************************************
ok: [192.168.201.16]

TASK [# TASK 02 # - Apply and Reload App Protect Policy] *************************************************************
changed: [192.168.201.16]

PLAY RECAP ***********************************************************************************************************
192.168.201.16             : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

