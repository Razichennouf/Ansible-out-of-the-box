# Ansible-out-of-the-box
<!DOCTYPE html>

<h1> Ansible Tip: Inserting After Last Occurrence</h1>

<h2>Ansible Tip: Inserting Content After the Last Occurrence of a Specific Line</h2>
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

<p><strong>Note:</strong> Always back up your file before running this playbook, especially on production environments.</p>
