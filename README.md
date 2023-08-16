# Ansible-out-of-the-box
<!DOCTYPE html>

<h2>Tip 1 : Inserting Content After the Last Occurrence of a Specific Line</h2>
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
<h2> Tip 1 :  Changing elastic user password and saving it in the current directory for later use in 400 mode, With an API call use case  </h1>
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
<p><strong>Note:</strong> Always back up your file before running this playbook, especially on production environments.</p>
