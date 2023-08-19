# Ansible-out-of-the-box
<!DOCTYPE html>

<h2>Tip 1: Inserting Content After the Last Occurrence of a Specific Line</h2>
<p>This technique shows how to insert content after the last occurrence of a particular line in a file using Ansible. We employ a combination of the <code>slurp</code> and <code>set_fact</code> modules to compute the insertion point, then use the <code>copy</code> module to write the modified content back.</p>

<h3>Scenario:</h3>
<p>Given a file that contains multiple occurrences of a specific line (e.g., <code>} # managed by Certbot</code>), the goal is to insert a new block of content after the last occurrence of this line.</p>

<pre>
<code>
---
- hosts: your_target_host
  vars:
    nginx_conf_path: /path/to/your/nginx/conf/file.conf

  tasks:
    - name: Slurp the nginx config content
      slurp:
        src: "{{ nginx_conf_path }}"
      register: slurp_nginx_conf

    - name: Convert the nginx config content from base64 to string
      set_fact:
        nginx_conf_content: "{{ slurp_nginx_conf.content | b64decode }}"

    - name: Find position of the last '} # managed by Certbot' in the reversed nginx config content
      set_fact:
        reversed_position: "{{ nginx_conf_content[::-1].find('tobtreC yb deganam # }') }}"

    - name: Calculate the position in the non-reversed nginx config content
      set_fact:
        insert_position: "{{ nginx_conf_content|length - reversed_position - 1 }}"

    - name: Get all content up to the insertion point
      set_fact:
        nginx_conf_before_insert: "{{ nginx_conf_content[:insert_position] }}"

    - name: Get all content after the insertion point
      set_fact:
        nginx_conf_after_insert: "{{ nginx_conf_content[insert_position:] }}"

    - name: Construct new nginx config with inserted block
      set_fact:
        nginx_new_content: "{{ nginx_conf_before_insert }}\n# Your Block Content Here\n{{ nginx_conf_after_insert }}"

    - name: Write the new nginx config back to the file
      copy:
        content: "{{ nginx_new_content }}"
        dest: "{{ nginx_conf_path }}"
</code>
</pre>
<h2> Tip 2:  Changing elastic user password and saving it in the current directory for later use in 400 mode, With an API call use case  </h1>
<pre>
<code>
---
#
- name: Chaning elastic password
  shell: |
     yes | sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u test | grep -oP 'New >
     chmod 400 elastic_pw
  #'/New value:/ {print $NF}' output.txt
  #sed -n 's/New value: //p' output.txt
- name: retrieve the elastic_pw value
  slurp:
     src: /home/ubuntu/elastic_pw
  register: elastic_pw_val
- name: Setting up number_of_replicas to 0 via API call
  shell: |
       curl -k -H "Content-Type: application/json" -XPUT https://localhost:9200/*/_settings -d '{ "index" : { "number_of_replicas" : 0 } }' -u elastic:"{{ elastic_pw_val.content | b64decode }}"
</code>
</pre>

<h2>Tip 3: Dynamic Inventory Creation from a Database</h2>
<h3>Scenario:</h3>
You have a MySQL database where details of hosts (like IP addresses, hostnames, and roles) are stored. This might be because you have a dynamic cloud environment, where VMs or containers are constantly being created or destroyed, and a system to log these to a database.
</p>

<h3>Objective:</h3>
<p>
You want to fetch these details from the database and use them to create an Ansible dynamic inventory. This allows you to manage these hosts using Ansible playbooks without having to constantly update a static inventory file.
</p>

<h3>Steps:</h3>

<ol>
    <li><strong>Set Up a MySQL Database:</strong>
        <ul>
            <li>Ensure you have a MySQL server running.</li>
            <li>Create a table structure to store host details, e.g.:
            <pre>
<code>
CREATE TABLE hosts (
    id INT PRIMARY KEY,
    hostname VARCHAR(255),
    ip_address VARCHAR(255),
    role VARCHAR(255)
);
</code>
            </pre>
            </li>
        </ul>
    </li>

  <li><strong>Fetch Host Details from the Database:</strong>
        <ul>
            <li>Use the <code>community.general.mysql_query</code> Ansible module to fetch data:
            <pre>
<code>
- name: Fetch host details from MySQL
  community.general.mysql_query:
    login_host: "your_database_host"
    login_user: "your_db_user"
    login_password: "your_db_password"
    database: "your_database_name"
    query: SELECT hostname, ip_address, role FROM hosts
  register: query_result
</code>
            </pre>
            </li>
        </ul>
    </li>

   <li><strong>Parse the Fetched Data to Create an Inventory:</strong>
        <ul>
            <li>Process the query results to create a dictionary structure suitable for an Ansible inventory. The <code>set_fact</code> module can help with this.</li>
        </ul>
    </li>

  <li><strong>Use the Dynamic Inventory:</strong>
        <ul>
            <li>Once you've structured the data correctly, you can:
                <ul>
                    <li>Write it to a JSON or YAML file and use it as a dynamic inventory source in Ansible.</li>
                    <li>Directly utilize the constructed data structure in subsequent tasks.</li>
                </ul>
            </li>
            <li>Example of how to run a playbook with the generated inventory:
            <pre><code>ansible-playbook -i dynamic_inventory.json your_playbook.yml</code></pre>
            </li>
        </ul>
    </li>
</ol>
<h2>Tip 3: Dynamic keystore creation and changing values inside kibana.yml </h2>
<h3>Scenario: </h3>
<code>
#- name: Create Kibana Keystore
#  command: "/usr/share/kibana/bin/kibana-keystore create"
#  args:
#      creates: /etc/kibana/kibana.keystore
#  register: keystore_created
#  changed_when: keystore_created.rc == 0  # Mark as changed only if keystore was created
#- name: Add Value to Kibana Keystore
#  command: "/usr/share/kibana/bin/kibana-keystore add ES_PWD"
#  args:
#        stdin: "{{ elastic_pw_val }}\n"  # Provide the ES_PWD value from Ansible variable
#   when: keystore_created.changed  # Only execute if keystore was created
#- name: Confirm Keystore Overwrite
#  command: "/bin/bash"
#  args:
#        stdin: "Y\n"  # Automatically answer 'Y' to overwrite prompt
#  when: keystore_created.changed  # Only execute if keystore was created
#- name: Appending the elasticsearch credentials in kibana config file
#  lineinfile:
#      path: /etc/kibana/kibana.yml
#      regex: '^#elasticsearch.username:'
#      line: 'elasticsearch.username: "elastic"'
#- name: Appending the elasticsearch credentials in kibana config file
#  lineinfile:
#      path: /etc/kibana/kibana.yml
#      regex: '^#elasticsearch.password'
#      line: 'elasticsearch.username: "${es_pwd}"'
</code>

<h3>Notes:</h3>
<ul>
    <li>The advantage of this setup is that your Ansible inventory will always reflect the actual state of your infrastructure as recorded in the database.</li>
    <li>This scenario uses MySQL as an example, but it can be adapted for other databases like PostgreSQL, SQLite, etc. by using the appropriate Ansible modules.</li>
    <li>Consider error-handling strategies. What should the playbook do if it can't connect to the database, or if the returned data doesn't match the expected format?</li>
    <li>Ensure you secure your database credentials. Using Ansible Vault to encrypt sensitive data is recommended.</li>
</ul>

<p>This is a high-level overview of the scenario. Actual implementation might vary based on specific database structures, desired inventory formats, and other unique needs.</p>


<p><strong>Note:</strong> Always back up your file before running this playbook, especially on production environments.</p>
