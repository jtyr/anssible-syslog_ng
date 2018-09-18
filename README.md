syslog_ng
=========

Ansible role which helps to install and configure `syslog-ng`.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```yaml
---

- name: Default usage
  hosts: all
  roles:
    - syslog_ng

- name: Amend the default configuration
  hosts: all
  vars:
    syslog_ng_config:
      # Change 'use_dns' to 'yes'
      syslog_ng_config_options_use_dns: use_dns (yes)
      # Add option
      syslog_ng_config_options__custom:
        - create_dirs (yes)
      # Add source
      syslog_ng_config_sources__custom:
        - source s_remote:
            - udp(ip(0.0.0.0) port(514))
      # Add destination
      syslog_ng_config_destinations__custom:
        - destination d_test:
            - file("/var/log/mylogs/$PROGRAM.log")
      # Add log
      syslog_ng_config_logs__custom:
        - log:
            - source(s_remote)
            - destination(d_test)
      # Add custom config (additional include)
      syslog_ng_config__custom:
        - '@include "/etc/syslog-ng/conf.d/*.conf"'
  roles:
    - syslog_ng

- name: Write configuration from scratch
  hosts: all
  vars:
    syslog_ng_config:
      - "@version:3.5!;"
      - '@include "scl.conf"!;'
      - options:
          - flush_lines (0)
          - time_reopen (10)
          - log_fifo_size (1000)
          - chain_hostnames (off)
          - use_dns (no)
          - use_fqdn (no)
          - create_dirs (yes)
          - keep_hostname (yes)
      - source s_remote:
          - udp(ip(0.0.0.0) port(514))
      - destination d_test:
          - file("/var/log/mylogs/$PROGRAM.log")
      - log:
          - source(s_remote)
          - destination(d_test)
  roles:
    - syslog_ng
```

The default configuration is the following:

```nginx
@version:3.5;
@include "scl.conf";

options {
  flush_lines (0);
  time_reopen (10);
  log_fifo_size (1000);
  chain_hostnames (off);
  use_dns (no);
  use_fqdn (no);
  create_dirs (no);
  keep_hostname (yes);
};

source s_sys {
  system();
};

destination d_cons {
  file("/dev/console");
};

destination d_mesg {
  file("/var/log/messages");
};

destination d_auth {
  file("/var/log/secure");
};

destination d_mail {
  file("/var/log/maillog" flush_lines(10));
};

destination d_spol {
  file("/var/log/spooler");
};

destination d_boot {
  file("/var/log/boot.log");
};

destination d_cron {
  file("/var/log/cron");
};

destination d_kern {
  file("/var/log/kern");
};

destination d_mlal {
  usertty("*");
};

filter f_kernel {
  facility(kern);
};

filter f_default {
  level(info..emerg) and not (facility(mail) or facility(authpriv) or facility(cron));
};

filter f_auth {
  facility(authpriv);
};

filter f_mail {
  facility(mail);
};

filter f_emergency {
  level(emerg);
};

filter f_news {
  facility(uucp) or (facility(news) and level(crit..emerg));
};

filter f_boot {
  facility(local7);
};

filter f_cron {
  facility(cron);
};

log {
  source(s_sys);
  filter(f_kernel);
  destination(d_kern);
};

log {
  source(s_sys);
  filter(f_default);
  destination(d_mesg);
};

log {
  source(s_sys);
  filter(f_auth);
  destination(d_auth);
};

log {
  source(s_sys);
  filter(f_mail);
  destination(d_mail);
};

log {
  source(s_sys);
  filter(f_emergency);
  destination(d_mlal);
};

log {
  source(s_sys);
  filter(f_news);
  destination(d_spol);
};

log {
  source(s_sys);
  filter(f_boot);
  destination(d_boot);
};

log {
  source(s_sys);
  filter(f_cron);
  destination(d_cron);
};
```


Role variables
--------------

Variables used by the role:

```yaml
# Package to install (you can force certain version here)
syslog_ng_pkg: syslog-ng

# Package to install (you can force certain version here)
syslog_ng_service: syslog-ng

# Whether to install EPEL YUM repo
syslog_ng_epel_install: "{{ yumrepo_epel_install | default(true) }}"

# EPEL YUM repo URL
syslog_ng_epel_yumrepo_url: "{{ yumrepo_epel_url | default('https://dl.fedoraproject.org/pub/epel/$releasever/$basearch/') }}"

# EPEL YUM repo GPG key
syslog_ng_epel_yumrepo_gpgkey: "{{ yumrepo_epel_gpgkey | default('https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever') }}"

# Additional EPEL params
syslog_ng_epel_yumrepo_params: "{{ yumrepo_epel_params | default({}) }}"

# Path to the main config file
syslog_ng_config_file: /etc/syslog-ng/syslog-ng.conf


# Value of the cersion and top level import
syslog_ng_config_version: "3.5"
syslog_ng_config_include_scl: scl.conf


# Values of the options of the options section
syslog_ng_config_options_flush_lines: flush_lines (0)
syslog_ng_config_options_time_reopen: time_reopen (10)
syslog_ng_config_options_log_fifo_size: log_fifo_size (1000)
syslog_ng_config_options_chain_hostnames: chain_hostnames (off)
syslog_ng_config_options_use_dns: use_dns (no)
syslog_ng_config_options_use_fqdn: use_fqdn (no)
syslog_ng_config_options_create_dirs: create_dirs (no)
syslog_ng_config_options_keep_hostname: keep_hostname (yes)

# Default options of the options section
syslog_ng_config_options__default:
  - "{{ syslog_ng_config_options_flush_lines }}"
  - "{{ syslog_ng_config_options_time_reopen }}"
  - "{{ syslog_ng_config_options_log_fifo_size }}"
  - "{{ syslog_ng_config_options_chain_hostnames }}"
  - "{{ syslog_ng_config_options_use_dns }}"
  - "{{ syslog_ng_config_options_use_fqdn }}"
  - "{{ syslog_ng_config_options_create_dirs }}"
  - "{{ syslog_ng_config_options_keep_hostname }}"

# Custom options of the options section
syslog_ng_config_options__custom: []

# Final options of the options section
syslog_ng_config_options: "{{
  syslog_ng_config_options__default +
  syslog_ng_config_options__custom }}"


# Structure of the s_sys source
syslog_ng_config_sources_s_sys:
  source s_sys:
    - system()

# Default list of the sources
syslog_ng_config_sources__default:
  - "{{ syslog_ng_config_sources_s_sys }}"

# Custom list of the sources
syslog_ng_config_sources__custom: []

# Final list of the sources
syslog_ng_config_sources: "{{
  syslog_ng_config_sources__default +
  syslog_ng_config_sources__custom }}"


# Structures of the individual destinations
syslog_ng_config_destinations_d_cons:
  destination d_cons:
    - file("/dev/console")
syslog_ng_config_destinations_d_mesg:
  destination d_mesg:
    - file("/var/log/messages")
syslog_ng_config_destinations_d_auth:
  destination d_auth:
    - file("/var/log/secure")
syslog_ng_config_destinations_d_mail:
  destination d_mail:
    - file("/var/log/maillog" flush_lines(10))
syslog_ng_config_destinations_d_spol:
  destination d_spol:
    - file("/var/log/spooler")
syslog_ng_config_destinations_d_boot:
  destination d_boot:
    - file("/var/log/boot.log")
syslog_ng_config_destinations_d_cron:
  destination d_cron:
    - file("/var/log/cron")
syslog_ng_config_destinations_d_kern:
  destination d_kern:
    - file("/var/log/kern")
syslog_ng_config_destinations_d_mlal:
  destination d_mlal:
    - usertty("*")

# Default list of the destinations
syslog_ng_config_destinations__default:
  - "{{ syslog_ng_config_destinations_d_cons }}"
  - "{{ syslog_ng_config_destinations_d_mesg }}"
  - "{{ syslog_ng_config_destinations_d_auth }}"
  - "{{ syslog_ng_config_destinations_d_mail }}"
  - "{{ syslog_ng_config_destinations_d_spol }}"
  - "{{ syslog_ng_config_destinations_d_boot }}"
  - "{{ syslog_ng_config_destinations_d_cron }}"
  - "{{ syslog_ng_config_destinations_d_kern }}"
  - "{{ syslog_ng_config_destinations_d_mlal }}"

# Custom list of the destinations
syslog_ng_config_destinations__custom: []

# Final list of the destinations
syslog_ng_config_destinations: "{{
  syslog_ng_config_destinations__default +
  syslog_ng_config_destinations__custom }}"


# Structures of the individual filters
syslog_ng_config_filters_f_kernel:
  filter f_kernel:
    - facility(kern)
syslog_ng_config_filters_f_default:
  filter f_default:
    - level(info..emerg) and not (facility(mail) or facility(authpriv) or facility(cron))
syslog_ng_config_filters_f_auth:
  filter f_auth:
    - facility(authpriv)
syslog_ng_config_filters_f_mail:
  filter f_mail:
    - facility(mail)
syslog_ng_config_filters_f_emergency:
  filter f_emergency:
    - level(emerg)
syslog_ng_config_filters_f_news:
  filter f_news:
    - facility(uucp) or (facility(news) and level(crit..emerg))
syslog_ng_config_filters_f_boot:
  filter f_boot:
    - facility(local7)
syslog_ng_config_filters_f_cron:
  filter f_cron:
    - facility(cron)

# Default list of the filters
syslog_ng_config_filters__default:
  - "{{ syslog_ng_config_filters_f_kernel }}"
  - "{{ syslog_ng_config_filters_f_default }}"
  - "{{ syslog_ng_config_filters_f_auth }}"
  - "{{ syslog_ng_config_filters_f_mail }}"
  - "{{ syslog_ng_config_filters_f_emergency }}"
  - "{{ syslog_ng_config_filters_f_news }}"
  - "{{ syslog_ng_config_filters_f_boot }}"
  - "{{ syslog_ng_config_filters_f_cron }}"

# Custom list of the filters
syslog_ng_config_filters__custom: []

# Final list of the filters
syslog_ng_config_filters: "{{
  syslog_ng_config_filters__default +
  syslog_ng_config_filters__custom }}"


# Structures of the individual logs
syslog_ng_config_logs_s_sys_f_kernel_d_kern:
  log:
    - source(s_sys)
    - filter(f_kernel)
    - destination(d_kern)
syslog_ng_config_logs_s_sys_f_default_d_mesg:
  log:
    - source(s_sys)
    - filter(f_default)
    - destination(d_mesg)
syslog_ng_config_logs_s_sys_f_auth_d_auth:
  log:
    - source(s_sys)
    - filter(f_auth)
    - destination(d_auth)
syslog_ng_config_logs_s_sys_f_mail_d_mail:
  log:
    - source(s_sys)
    - filter(f_mail)
    - destination(d_mail)
syslog_ng_config_logs_s_sys_f_emergency_d_mlal:
  log:
    - source(s_sys)
    - filter(f_emergency)
    - destination(d_mlal)
syslog_ng_config_logs_s_sys_f_news_d_spol:
  log:
    - source(s_sys)
    - filter(f_news)
    - destination(d_spol)
syslog_ng_config_logs_s_sys_f_boot_d_boot:
  log:
    - source(s_sys)
    - filter(f_boot)
    - destination(d_boot)
syslog_ng_config_logs_s_sys_f_cron_d_cron:
  log:
    - source(s_sys)
    - filter(f_cron)
    - destination(d_cron)

# Default list of the logs
syslog_ng_config_logs__default:
  - "{{ syslog_ng_config_logs_s_sys_f_kernel_d_kern }}"
  - "{{ syslog_ng_config_logs_s_sys_f_default_d_mesg }}"
  - "{{ syslog_ng_config_logs_s_sys_f_auth_d_auth }}"
  - "{{ syslog_ng_config_logs_s_sys_f_mail_d_mail }}"
  - "{{ syslog_ng_config_logs_s_sys_f_emergency_d_mlal }}"
  - "{{ syslog_ng_config_logs_s_sys_f_news_d_spol }}"
  - "{{ syslog_ng_config_logs_s_sys_f_boot_d_boot }}"
  - "{{ syslog_ng_config_logs_s_sys_f_cron_d_cron }}"

# Custom list of the logs
syslog_ng_config_logs__custom: []

# Final list of the logs
syslog_ng_config_logs: "{{
  syslog_ng_config_logs__default +
  syslog_ng_config_logs__custom }}"


# Default config
syslog_ng_config__default:
  - "@version:{{ syslog_ng_config_version }}"
  - '@include "{{ syslog_ng_config_include_scl }}"'
  - options: "{{ syslog_ng_config_options }}"

# Custom config
syslog_ng_config_custom: []

# Final config
syslog_ng_config: "{{
  syslog_ng_config__default +
  syslog_ng_config_sources +
  syslog_ng_config_destinations +
  syslog_ng_config_filters +
  syslog_ng_config_logs +
  syslog_ng_config_custom }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`elasticsearch`](https://github.com/jtyr/ansible-elasticsearch) (optional)
- [`filebeat`](https://github.com/jtyr/ansible-filebeat) (optional)
- [`kibana`](https://github.com/jtyr/ansible-kibana) (optional)
- [`logstash`](https://github.com/jtyr/ansible-logstash) (optional)


License
-------

MIT


Author
------

Jiri Tyr
