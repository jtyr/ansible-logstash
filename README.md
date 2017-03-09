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
    # Change the JVM heap size
    logstash_sysconfig:
      es_heap_size: 512m
    logstash_jvm_options_x_ms: 256m
    logstash_jvm_options_x_mx: 512m
    # Bind to all network interfaces
    logstash_config_http_data: 0.0.0.0
    # Change the port number range
    logstash_config_http__custom:
      port: 9800-9900
    # Customize the main Logstash config
    logstashs_config__custom:
      # Increase the number of workers
      pipeline:
        workers: 3
    # Create events
    logstash_pipeline:
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

# YUM repo URL
logstash_yum_repo_url: https://artifacts.elastic.co/packages/5.x/yum

# YUM repo GPG key
logstash_yum_repo_key: https://artifacts.elastic.co/GPG-KEY-elasticsearch

# Extra EPEL YUM repo params
logstash_yum_repo_params: {}

# Name of the service
logstash_service: logstash


# Path to the main config file
logstash_config_file: /etc/logstash/logstash.yml

# Values of the default http part of the Logstash config
logstash_config_http_host: 127.0.0.1

# Default http part of the Logstash config
logstash_config_http__default:
    host: "{{ logstash_config_http_host }}"

# Custom http part of the Logstash config
logstash_config_http__custom: {}

# Final http part of the Logstash config
logstash_config_http: "{{
  logstash_config_http__default.update(logstash_config_http__custom) }}{{
  logstash_config_http__default }}"

# Values of the default host part of the Logstash config
logstash_config_host_data: /var/lib/logstash
logstash_config_host_config: /etc/logstash/conf.d
logstash_config_host_logs: /var/log/logstash

# Default host part of the Logstash config
logstash_config_host__default:
    data: "{{ logstash_config_host_data }}"
    config: "{{ logstash_config_host_config }}"
    logs: "{{ logstash_config_host_logs }}"

# Custom host part of the Logstash config
logstash_config_host__custom: {}

# Final host part of the Logstash config
logstash_config_host: "{{
  logstash_config_host__default.update(logstash_config_host__custom) }}{{
  logstash_config_host__default }}"

# Default Logstash config
logstash_config__default:
  http: "{{ logstash_config_http }}"
  path: "{{ logstash_config_host }}"

# Custom Logstash config
logstash_config__custom: {}

# Final Logstash config
logstash_config: "{{
  logstash_config__default.update(logstash_config__custom) }}{{
  logstash_config__default }}"


# Path to the Logstash config directory
logstash_pipeline_path: /etc/logstash/conf.d

# Default configuration
logstash_pipeline__default: {}

# Custom configuration
logstash_pipeline__custom: {}

# Final configuration
logstash_pipeline: "{{
  logstash_pipeline__default.update(
  logstash_pipeline__custom) }}{{
  logstash_pipeline__default }}"


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


# Location of the jvm.options file
logstash_jvm_options_file: /etc/logstash/jvm.options

# Values of the default JVM system properties
logstash_jvm_options_d_file_encoding: UTF-8
logstash_jvm_options_d_java_awt_headless: "true"

# Default JVM system properties
logstash_jvm_options_d__default:
  - file.encoding={{ logstash_jvm_options_d_file_encoding }}
  - java.awt.headless={{ logstash_jvm_options_d_java_awt_headless }}

# Custom JVM system properties
logstash_jvm_options_d__custom: []

# Final JVM system properties
logstash_jvm_options_d: "{{
  logstash_jvm_options_d__default +
  logstash_jvm_options_d__custom }}"

# Values of the default JVM non-standard options
logstash_jvm_options_x_ms: 256m
logstash_jvm_options_x_mx: 1g
logstash_jvm_options_x_X_CMSInitiatingOccupancyFraction: 75
logstash_jvm_options_x_X_DisableExplicitGC: yes
logstash_jvm_options_x_X_HeapDumpOnOutOfMemoryError: yes
logstash_jvm_options_x_X_UseCMSInitiatingOccupancyOnly: yes
logstash_jvm_options_x_X_UseConcMarkSweepGC: yes
logstash_jvm_options_x_X_UseParNewGC: yes

# Default JVM non-standard options
logstash_jvm_options_x__default:
  - ms{{ logstash_jvm_options_x_ms }}
  - mx{{ logstash_jvm_options_x_mx }}
  - X:CMSInitiatingOccupancyFraction={{ logstash_jvm_options_x_X_CMSInitiatingOccupancyFraction }}
  - X:{{ '+' if logstash_jvm_options_x_X_DisableExplicitGC else '-' }}DisableExplicitGC
  - X:{{ '+' if logstash_jvm_options_x_X_HeapDumpOnOutOfMemoryError else '-' }}HeapDumpOnOutOfMemoryError
  - X:{{ '+' if logstash_jvm_options_x_X_UseCMSInitiatingOccupancyOnly else '-' }}UseCMSInitiatingOccupancyOnly
  - X:{{ '+' if logstash_jvm_options_x_X_UseConcMarkSweepGC else '-' }}UseConcMarkSweepGC
  - X:{{ '+' if logstash_jvm_options_x_X_UseParNewGC else '-' }}UseParNewGC

# Custom JVM non-standard options
logstash_jvm_options_x__custom: []

# Final JVM non-standard options
logstash_jvm_options_x: "{{
  logstash_jvm_options_x__default +
  logstash_jvm_options_x__custom }}"

# Default JVM options
logstash_jvm_options__default:
  -D: "{{ logstash_jvm_options_d }}"
  -X: "{{ logstash_jvm_options_x }}"

# Custom JVM options
logstash_jvm_options__custom: {}

# Final JVM options
logstash_jvm_options: "{{
  logstash_jvm_options__default.update(logstash_jvm_options__custom) }}{{
  logstash_jvm_options__default }}"


# Location of the jvm.options file
logstash_startup_options_file: /etc/logstash/startup.options

# Values of the default startup config options
logstash_startup_options_javacmd: /usr/bin/java
logstash_startup_options_ls_home: /usr/share/logstash
logstash_startup_options_ls_settings_dir: /etc/logstash
logstash_startup_options_ls_opts: --path.settings {{ logstash_startup_options_ls_settings_dir }}
logstash_startup_options_ls_java_opts: ""
logstash_startup_options_ls_pidfile: /var/run/logstash.pid
logstash_startup_options_ls_user: logstash
logstash_startup_options_ls_group: logstash
logstash_startup_options_ls_gc_log_file: /var/log/logstash/gc.log
logstash_startup_options_ls_open_files: 16384
logstash_startup_options_ls_nice: 19
logstash_startup_options_service_name: logstash
logstash_startup_options_service_description: logstash

# Default startup options
logstash_startup_options__default:
  javacmd: "{{ logstash_startup_options_javacmd }}"
  ls_home: "{{ logstash_startup_options_ls_home }}"
  ls_settings_dir: "{{ logstash_startup_options_ls_settings_dir }}"
  ls_opts: "{{ logstash_startup_options_ls_opts }}"
  ls_java_opts: "{{ logstash_startup_options_ls_java_opts }}"
  ls_pidfile: "{{ logstash_startup_options_ls_pidfile }}"
  ls_user: "{{ logstash_startup_options_ls_user }}"
  ls_group: "{{ logstash_startup_options_ls_group }}"
  ls_gc_log_file: "{{ logstash_startup_options_ls_gc_log_file }}"
  ls_open_files: "{{ logstash_startup_options_ls_open_files }}"
  ls_nice: "{{ logstash_startup_options_ls_nice }}"
  service_name: "{{ logstash_startup_options_service_name }}"
  service_description: "{{ logstash_startup_options_service_description }}"

# Custom startup options
logstash_startup_options__custom: {}

# Final startup options
logstash_startup_options: "{{
  logstash_startup_options__default.update(logstash_startup_options__custom) }}{{
  logstash_startup_options__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`elasticsearch`](https://github.com/jtyr/ansible-elasticsearch) (optional)
- [`filebeat`](https://github.com/jtyr/ansible-filebeat) (optional)
- [`kibana`](https://github.com/jtyr/ansible-kibana) (optional)
- [`oracle-java`](https://github.com/jtyr/ansible-oracle_java) (optional)


License
-------

MIT


Author
------

Jiri Tyr
