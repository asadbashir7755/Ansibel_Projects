what are roles
why roles
role anatomy
role folder structure for production env
precedence of variables
discuss everthing in  roles like templates,vars etc
in role where keep inventory where keep cfg file where variables
how generate roles 
what are nested roles how use them
dependencies: [default-system-conf]
like this i think
it will first run dependency then next like this TASK [default_system_conf : create a deploy_user group] ************************************************************************************
ok: [13.233.225.159]

TASK [default_system_conf : create a deploy user with uid 5004] ****************************************************************************
ok: [13.233.225.159]

other things about roles
also loops in ansible 
syntax 
- name: Create multiple directories
  ansible.builtin.file:
    path: "/opt/{{ item }}"
    state: directory
  loop:
    - app1
    - app2
    - app3
with_dict
- name: Add users with comments
  ansible.builtin.user:
    name: "{{ item.key }}"
    comment: "{{ item.value }}"
  with_dict:
    asad: "DevOps Engineer"
    saqib: "Frontend Developer"

with_fileglob
- name: Copy all HTML files
  ansible.builtin.copy:
    src: "{{ item }}"
    dest: /var/www/html/
  with_fileglob:
    - "*.html"

with_nested
- name: Create user and group combinations
  debug:
    msg: "User {{ item[0] }} belongs to group {{ item[1] }}"
  with_nested:
    - [ 'asad', 'saqib' ]
    - [ 'devops', 'frontend' ]

with sequence
- name: Create numbered directories
  ansible.builtin.file:
    path: "/data/{{ item }}"
    state: directory
  with_sequence: start=1 end=5

websites:
  - name: site1
    port: 8081
    doc_root: /var/www/site1
  - name: site2
    port: 8082
    doc_root: /var/www/site2
for templates 
{% for site in websites %}
<VirtualHost *:{{ site.port }}>
    ServerAdmin webmaster@localhost
    DocumentRoot {{ site.doc_root }}
    ServerName {{ site.name }}
    ErrorLog ${APACHE_LOG_DIR}/{{ site.name }}-error.log
    CustomLog ${APACHE_LOG_DIR}/{{ site.name }}-access.log combined
</VirtualHost>
{% %}
variable precedence in ansible in roles
hash behaviour in ansible 
replace is default 
we can use merge 
add example

how hanle errors usig block and rescue  : why use it
ignore_error : yes why use it
how use registers in ansible 