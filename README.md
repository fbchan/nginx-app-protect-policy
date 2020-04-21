# NGINX App Protect Ansible Playbook
1. nap_play.yml --> Generate app protect policy locally from jinja2 template (nap_base.j2) and apply onto remote NGINX (via ansible copy module) and reload nginx.
2. elk_query_sig.yml --> Query elasticsearch (unauthenticated) to return signature_id by providing support_id

## nap_play.yml
Only require to define configuration object in nap_var.yml. Currently only build app protect policy based on those variables. Can be easily extended to includes other variable.

## Example

Note: You may need to update inventory file to specify your server
```
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
```

### Sample Generated App Protect Policy
```
{
  "name": "waf-policy-dmz",
  "template": {
    "name": "POLICY_TEMPLATE_NGINX_BASE"
  },
  "applicationLanguage": "utf-8",
  "server-technologies": [
   {
     "serverTechnologyName": "Unix/Linux"
   },
   {
     "serverTechnologyName": "PHP"
   }
  ],
  "signature-settings":{
        "signatureStaging": false
  },
  "modifications":[

  ],
  "enforcementMode": "blocking",
  "signatures": [
  {
      "signatureId": 200001475,
      "enabled": false
  },
  {
      "signatureId": 200000098,
      "enabled": false
  },
  {
      "signatureId": 200001088,
      "enabled": false
  }
  ],
  "blocking-settings": {
   "violations": [
    {
      "name": "VIOL_HTTP_PROTOCOL",
      "alarm": true,
      "block": false
    },
    {
      "name": "VIOL_PARAMETER_VALUE_METACHAR",
      "alarm": false,
      "block": false
    },
    {
      "name": "VIOL_EVASION",
      "alarm": true,
      "block": false
    }
  ]
  },
  "signature-sets": [
    {
            "name": "High Accuracy Signatures",
            "block": true,
            "alarm": true
    }
  ]
}
```

### elk_query_sig.yml
In the event app protect rejected a URL provided with support ID.

Note: You may need to update Elasticsearch IP to your ELK stack.

Example App Protect reject page
```
The requested URL was rejected. Please consult with your administrator.

Your support ID is: 9825417222313866910

[Go Back]
```

```
$ ansible-playbook elk_query_sig.yml -e support_id=9825417222313866910
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [### PLAY 01 ### - Query Elasticsearch] ******************************************************************************************

TASK [# TASK 01 # - Query elasticsearch] **********************************************************************************************
ok: [localhost]

TASK [# TASK 02 # Print output] *******************************************************************************************************
ok: [localhost] => (item=Log Source: 'app-protec1.foobz.com.au'
Signature Name: '['XSS script tag (Headers)', 'XSS script tag end (Headers)', 'alert (Headers)']'
Signature IDs: '['200000097', '200000091', '200001089']'
) => {
    "msg": [
        "Log Source: 'app-protec1.foobz.com.au'",
        "Signature Name: '['XSS script tag (Headers)', 'XSS script tag end (Headers)', 'alert (Headers)']'",
        "Signature IDs: '['200000097', '200000091', '200001089']'",
        ""
    ]
}

PLAY RECAP ****************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

To perform signature IDs exception, add those signature IDs in nap_var.yml.

Example
```
server_tech:
- tech1:
  name: Unix/Linux
- tech2:
  name: PHP

enforementMode: blocking

exception_signature:
 - id: 200000097
 - id: 200000091
 - id: 200001089


block_violation:
- violation1:
  name: VIOL_HTTP_PROTOCOL
  alarm_switch: true
  block_switch: false
- violation2:
  name: VIOL_PARAMETER_VALUE_METACHAR
  alarm_switch: false
  block_switch: false
- violation3:
  name: VIOL_EVASION
  alarm_switch: true
  block_switch: false

sig_set:
- sig_set1:
  name: High Accuracy Signatures
  alarm_switch: true
  block_switch: true
```