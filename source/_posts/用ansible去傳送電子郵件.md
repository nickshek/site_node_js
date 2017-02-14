---
title: 用ansible去傳送電子郵件
date: 2017-01-26
categories:
    - Ansible
---
Ansible在前幾年是一個非常熱門的開源項目，以我理解ansible其中一個功能是可用來部署應用程式，但是我個人覺得部署應用程式暫時有 capistrano或deployer-php也非常足夠，同時部署應用程式於多個伺服器也勝任有餘，總比手動部署來得有效率。

因此比起用Ansible來去試一試寫一個部署scripts，我想使用Ansible試一試一個簡單的例子去在各server 執行一些command (e.g df -h)，再把結果以電子郵件傳送，這樣也可以感受一下Ansible的威力，雖然個人比起編寫yaml (yaml 內或text template 可以使用 jinja2 template language，一個類似Django template engine的template engine)，個人更喜愛寫Ruby 或 PHP scripts 。

首先在/etc/ansible/ansible.cfg 設定 host_key_checking = False，這樣會令測試ansible時簡單一點，ansible.cfg 的範例如下所示:

```
[defaults]
host_key_checking = False
log_path=/path/to/log
```

之後建立一個ansible playbook e.g site.yml

```yaml
- hosts: all:!local
  tasks:
  - name: ping all hosts
    ping:
    register: ping_result
    ignore_errors: yes

  - name: check disk space for all hosts
    command: df -h
    register: disk_result
    ignore_errors: yes

  - name: check high cpu usage process
    shell: ps -eo pcpu,user,args | sort -r -k1 | head -n5
    register: cpu_result
    ignore_errors: yes

  - name: run ifconfig
    shell: ifconfig
    register: ifconfig_result
    ignore_errors: yes

  - name: template email
    local_action: template src=./files/email_template.j2 dest='./tmp/{{ hostvars[inventory_hostname]["inventory_hostname"] }}.txt'

- hosts: local
  tasks:
  - name: create result email
    template: src=./files/email_summary_template.j2 dest=./tmp/email.txt
    delegate_to: localhost

  - name: send email result
    local_action: mail host='{{ email_host }}' port='{{ email_port }}' to='{{ email_to }}' username='{{ email_username }}' password='{{ email_password }}' subject='Report' body='{{ lookup("file", "./tmp/email.txt") }}' from='{{ email_from }}' charset='UTF-8' subtype='html'

```

以上的yaml script的流程好簡單，純粹用ping + shell module在不同的server 去執行不同的指令，之後用register 去把執行結果紀錄至指定的variable。因為萬一執行時出現錯誤，我們還是希望ansible playbook可繼續執行，因此設定ignore_errors = true即可。

下一步便是把各servers的執行結果變為一個template，因為這個執行結果在local machine的下一個tasks是需要的，所以使用了local_action

以下便用 email_template.j2 的原始碼:

```
{% if ping_result is defined %}
<h2>Ping Command Result</h2>
-----------------------------------<br/>
{{ ping_result | to_nice_yaml | replace("\n","<br/>") }}
<br/>
{% endif %}

{% if disk_result is defined %}
<h2>Disk Command Result</h2>
-----------------------------------<br/>
{{ disk_result | to_nice_yaml | replace("\n","<br/>") }}
<br/>
{% endif %}

{% if cpu_result is defined %}
<h2>Command result for ps -eo pcpu,user,args | sort -r -k1 | head -n5</h2>
-----------------------------------<br/>
{{ cpu_result | to_nice_yaml | replace("\n","<br/>") }}
{% endif %}
<br/>
```

執行結果是一個python 的 dict，為了確保所有的內容都被列印出來，純粹用to_nice_yaml 列印出來即可。
一下步便是把每一個inventory的執行結果整理至一個email template，然後寄出。email_summary_template.j2的內容如下所示:

```
{% for host in groups['all'] %}
{% if host != 'localhost' %}
<h1>Result for
{{ hostvars[host]['inventory_hostname'] }}
</h1>
<b>==================================================================</b><br/>
{{lookup("file", "./tmp/" + hostvars[host]["inventory_hostname"] + ".txt")}}
{% endif %}
{% endfor %}
```

請注意，send email result的這個tasks全部都使用了variable，避免直接寫死任何容易更改設定在 playboook內，這一個動作明顯不符合Ansible本身的哲學，variable可以放在group_vars/all :

```
---
email_host: <email_host>
email_port: <email_port>
email_to: <email_to>
email_username: <email_username>
email_password: <email_password>
email_from: <email_from>

```

ansible的建議資料夾結構請參考 [http://docs.ansible.com/ansible/playbooks_best_practices.html#directory-layout](http://docs.ansible.com/ansible/playbooks_best_practices.html#directory-layout)

最後我們還是編寫hosts 檔案，以下是hosts的檔案範例，建議先設定ssh key來做認證，小心保管好private key + public key，避免把密碼寫在hosts file

```
[local]
localhost ansible_connection=local

[servers]
<example1.com> ansible_user=<example_user1> ansible_port=<example_port1> ansible_ssh_private_key_file=.ssh/id_rsa
<example1.com> ansible_user=<example_user2> ansible_port=<example_port2> ansible_ssh_private_key_file=.ssh/id_rsa
```

設定方法可參考 [https://blog.longwin.com.tw/2005/12/ssh_keygen_no_passwd/](https://blog.longwin.com.tw/2005/12/ssh_keygen_no_passwd/)

執行ansible playbook的方法相當簡單，如下所示:
```bash
ansible-playbook -i hosts site.yml
```

這樣我們便完成了一個好簡單的ansible playbook，這個playbook也有相當多的改進的地方，例如我們都把tasks 都放在site.yml，現在還好，沒有太多tasks。但沒有按roles去把不同的tasks放入去長遠都會有維護上的問題，site.yml愈簡單愈容易維護，另外一個可以改進的地方是lookup時的錯誤處理。

想知道這個project的檔案結構，可參考 [https://github.com/nickshek/ansible_mail_example](https://github.com/nickshek/ansible_mail_example)
