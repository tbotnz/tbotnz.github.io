# example calls

## postman

view over 80 examples of API calls to netpalm via the online postman collection [here](https://documenter.getpostman.com/view/2391814/T1DqgwcU?version=latest#33acdbb8-b5cd-4b55-bc67-b15c328d6c20)


## python
```
import requests

url = "127.0.0.1:9000/getconfig"

payload = "{\n\t\"library\": \"napalm\",\n\t\"connection_args\": {\n\t\t\"device_type\": \"cisco_ios\",\n\t\t\"host\": \"10.0.2.24\",\n\t\t\"username\": \"admin\",\n\t\t\"password\": \"admin\"\n\t},\n\t\"command\": \"show run | i hostname\",\n\t\"queue_strategy\": \"pinned\"\n}"
headers = {
  'x-api-key': '2a84465a-cf38-46b2-9d86-b84Q7d57f288',
  'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data = payload)

print(response.text.encode('utf8'))
```

## ansible

```
---
- hosts: localhost
  gather_facts: false
  vars:
    device_ip: device_ip_here
    device_username: username_here
    device_password: password_here
    netpalm_ip: netpalm_ip_here
    netpalm_port: 9000
    netpalm_api_key: api_key_here
  tasks:
    - name: Test netpalm from ansible
      uri:
        url: http://{{ netpalm_ip }}:{{ netpalm_port }}/getconfig
        headers:
          x-api-key: "{{ netpalm_api_key }}"
        return_content: true
        method: POST
        status_code: 200,201
        body_format: json
        body:
          {
            "library": "napalm",
            "connection_args": {
              "device_type": "cisco_ios",
              "host": "{{ device_ip }}",
              "username": "{{ device_username }}",
              "password": "{{ device_password }}"
            },
            "command": "show version",
            "args": {
              "use_textfsm": true
            },
            "webhook": {
              "name": "default_webhook",
              "args": {
                "netpalm": "is neat"
              }
            },
           "queue_strategy": "fifo"
         }
      register: netpalm_response
    - name: setting a fact for the task_id
      set_fact:
        task_id: "{{ netpalm_response.json.data.task_id }}"
    - name: gather results
      uri:
        url: http://{{ netpalm_ip }}:{{ netpalm_port }}/task/{{ task_id }}
        headers:
          x-api-key: "{{ netpalm_api_key }}"
        return_content: true
        method: GET
        status_code: 200,201
        body_format: json
      register: netpalm_task_results
      until: netpalm_task_results.json.data.task_status == "finished"
      retries: 10
      delay: 3
```