# ansible-role-zabbix-webscenario

Ansible role which creates or deletes (NO UPDATE) zabbix web scenarios
For each entry in the 'z_websites' variable with state: present, this role will :

* Add a web scenario
* Add a basic step
* Add a trigger associated to that web scenario

For each entry in the 'z_websites' variable with state: absent, this role will :
* Delete the web scenario associated to its name
* Delete the trigger associated to it

For any update of an existing web scenario, it must first be deleted (manually or by changing its state to absent, executing the playbook, then changing it back to 'present')

### Variables

Here is the list of all variables and their default values :
```yaml

# Authentification to API
z_user: api_ansible                                     #zabbix user to authenticate with API, must have access to the z_sourcehost
z_password: 'strongpassword'                            #zabbix user password
z_url: 'https://zabbix.mycompany.com/api_jsonrpc.php'   #zabbix URL

# Common variables to all webscenarios
z_sourcehost: "zabbixproxy01"                           #name of the sourcehost from where the web scenarios will be executed (must be already created)
z_applicationid: 1457                                   #application id of the application (category like) (must be already created)
z_interval: 30                                          #default interval of execution of the web scenarios
z_retries: 5                                            #default maximum number of retries
z_agent: 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.104 Safari/537.36' #useragent used by zabbix

# Definition of each website to monitor
z_websites:
    ## Creates a webscenario with two steps (one for http and one for https)
    ## First step checks http://mycompany.com using GET HTTP method, expects 200,301 or 302 as return code, expects string 'Welcome to mycompany website'
    ## Second step checks https://mycompany.com using GET HTTP method, expects 200,301 or 302 as return code, expects string 'Welcome to mycompany website'
    ## The associated trigger will have two tags, display and any_other_tag.
  - name: 'Our Company Website'                         #name of the web scenario
    url: 'mycompany.com'                                #url of the web scenario (DO NOT PUT http(s)://)
    state: present                                      #state, present or absent
    https: 'yes'                                        #is the website in https ? no by default
    method: 'GET'                                       #http method used, GET or HEAD, HEAD by default
    status_codes: '200,301,302'                         #expected http code returned, 200 by default
    required: 'Welcome to mycompany website'            #required string on the page, method MUST be GET for this to work
    auth:                                               #for websites that require basic HTTP authentication
      http_user: 'jim'                                  
      http_password: 'jimspassword'
    trigger_tags:                                       #tags to add with the associated trigger
      display: 'yes'
      any_other_tag: 'any_value'

    ## Creates a webscenario with one step, checking http://anotherwebsite.com/ using HEAD HTTP method, expects 200 as return code
  - name: 'Another website'
    url: 'anotherwebsite.com'
    state: present

    ## Other possible variables to put in z_websites
    monitored_from:                                     #attaches the web scenario on another host than z_sourcehost
      name: 'some_other_host'
      hostid: 4567
      applicationid: 1234
    http: 'no'                                          #for https-only websites, removes the http step
    ssl_verify_peer: 'no'                               #for https websites, disables the checking of the matching between the cn and the domain name of the website (default yes)
    ssl_verify_host: 'no'                               #for https websites, disables the checking of the root certificate against trusted store (default yes)

```

### Usage

Clone the repo.
```bash
$ git clone https://github.com/jdelvecchio/ansible-role-zabbix-webscenario
```
Then set vars in your playbook file.

Example playbook file :

```yaml
---
- hosts: 127.0.0.1
  gather_facts: no
  become: no
  roles:
    - role: ansible-role-zabbix-web-scenario
      z_user: api_ansible
      z_password: 'strongpassword'
    
      z_url: 'https://zabbix.mycompany.com/api_jsonrpc.php'
      z_sourcehost: "scasrvzbx01pp"
      z_applicationid: 1457
      z_interval: 30
      z_retries: 5
      z_agent: 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.104 Safari/537.36'

      z_websites:
        - name: 'Our Company Website'
          url: 'mycompany.com'
          state: present
          https: 'yes'
          method: 'GET'
          status_codes: '200,301,302'
          required: 'Welcome to mycompany website'
          trigger_tags:
            display: 'yes'
            any_other_tag: 'any_value'

        - name: 'Another website'
          url: 'anotherwebsite.com'
          state: present
```
