logstash
========

Ansible role which helps to install and configure Logstash.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```
---

- name: Example of how to install and configure Logstash
  hosts: all
  vars:
    # Change the starting parameters in the sysconfig file
    logstash_sysconfig:
      # Change the Java heap size
      es_heap_size: 300m
    logstash_config:
      # This is the file name which will be created in /etc/logstash/conf.d
      test1:
        # Here starts the Logstash configuration for that file
        - :input:
            - :file:
                path: /var/log/httpd/access_log
                start_position: beginning
        - :filter:
            - ':if [path] =~ "access"':
                - :mutate:
                    replace:
                      type: apache_access
                - :grok:
                    match:
                      message: "%{COMBINEDAPACHELOG}"
                - :date:
                    match:
                      - timestamp
                      - dd/MMM/yyyy:HH:mm:ss Z
            - ':else if [path] =~ "error"':
                - :mutate:
                    replace:
                      type: "apache_error"
            - :else:
                - :mutate:
                    replace:
                      type: "random_logs"
        - :output:
            - :elasticsearch:
                hosts:
                  - localhost:9200
            - :stdout:
                codec: rubydebug
  roles:
    - logstash
```


Role variables
--------------

```
# Package to be installed (explicit version can be specified here)
logstash_pkg: logstash

# Additional packages to be installed (e.g. Java)
logstash_additional_pkgs:
  - java

# Location of the config file
logstash_config_file: /etc/logstash/logstash.yml

# YUM repo URL
logstash_yum_repo_url: https://packages.elastic.co/logstash/2.4/centos

# YUM repo GPG key
logstash_yum_repo_key: https://packages.elastic.co/GPG-KEY-elasticsearch

# Extra EPEL YUM repo params
logstash_yum_repo_params: {}

# Name of the service
logstash_service: logstash


# Path to the logstash config directory
logstash_config_path: /etc/logstash/conf.d

# Default configuration
logstash_config__default: {}

# Custom configuration
logstash_config__custom: {}

# Final configuration
logstash_config: "{{
  logstash_config__default.update(
  logstash_config__custom) }}{{
  logstash_config__default }}"


# Path to the sysconfig file
logstash_sysconfig_file: /etc/sysconfig/logstash

logstash_sysconfig:
  kill_on_stop_timeout: 0
# Possible options:
#logstash_sysconfig:
#  javacmd: /usr/bin/java
#  ls_home: /var/lib/logstash
#  ls_opts: ""
#  ls_heap_size: 1g
#  ls_java_opts: "-Djava.io.tmpdir: $HOME"
#  ls_pidfile: /var/run/logstash.pid
#  ls_user: logstash
#  ls_log_file: /var/log/logstash/logstash.log
#  ls_use_gc_logging: "true"
#  ls_gc_log_file: /var/log/logstash/gc.log
#  ls_conf_dir: /etc/logstash/conf.d
#  ls_open_files: 16384
#  ls_nice: 19
#  kill_on_stop_timeout: 0
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`elasticsearch`](https://github.com/jtyr/ansible-elasticsearch) (optional)
- [`filebeat`](https://github.com/jtyr/ansible-filebeat) (optional)


License
-------

MIT


Author
------

Jiri Tyr
