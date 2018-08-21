# aspects_apache24
Configure Apache 2.4.

## Requirements

Set ```hash_behaviour=merge``` in your ansible.cfg file.

## Role Variables
This role overwrites the main Apache configuration file so that it will only include configuration managed by this role.

OS default configuration is left in place, but is NOT used.

> Note: Removing the OS default configuration was tried. Unfortunately, some module packages expect it to be present, and do not install correctly when it is not. 

### `aspects_apache24_enabled`
Flag to enable or disable this role. Default is disabled.

### `aspects_apache24_selinux_change`
Flag to enable or disable changing selinux port options. Default is False.

### `aspects_apache24_selinux_httpd_can_network_connect`
When `aspects_apache24_selinux_change` modify the `httpd_can_network_connect` SELinux boolean value. Set to true when True, set to false, when False. Default is True.

### `aspects_apache24_listen_ports`
A list of ports that selinux should allow apache to listen on.

### `aspects_apache24_test_configuration_root`
The directory that Ansible will write test configuration files to.

### `aspects_apache24_test_configuration_directories_list`
A list of directories that Ansible will make sure exists for test configuration.

### `aspects_apache24_configuration_directories_list`
A list of directories that Ansible will make sure exists for production configuration.

### `aspects_apache24_configuration_root`
The directory that contains the production configuration. I.E. `/etc/httpd` or `/etc/apache2`.

### `aspects_apache24_modules_root`
The directory that contains the binaries for any installed Apache module.

### `aspects_apache24_logs_root`
The directory that contains the log files. I.E. `/var/log/httpd` or `/var/log/apache2`.

### `aspects_apache24_base_modules`
A configuration block that contains the `LoadModule` lines required for Apache to run. The default block in [defaults/main.yml](defaults/main.yml) uses Jinja templating to ensure the appropriate modules load for the appropriate OS.

### `aspects_apache24_configuration_general`
A dictionary of general, non-vhost, non-module, Apache configuration. Each item represents a specific file in the `aspects.conf.d` directory. 

[defaults/main.yml](defaults/main.yml) contains the bare minimum configuration that worked on the Vagrant test boxes I used. Be sure to review it to make sure it works for your systems.

Use the following pattern:

```yaml
aspects_apache24_configuration_general:
  <key>:
    enabled: "<True|False>"
    weight: "<NN>"
    name: "<Name>"
    block: |
      <Block of configuration. Can use Jinja Templates.>
```

Each file is named like `<weight>-<name>.conf`. Apache will load the files in order according to their weight. Lower numbers load first.

You may use Jinja conditionals in the `enabled` value to enable or disable a block based on OS or some other condition.

### `aspects_apache24_configuration_modules`
A dictionary of module specific Apache configuration. Each item represents a specific file in the `aspects.mods.d` directory. 

Use the following pattern:

```yaml
aspects_apache24_configuration_modules:
  <key>:
    enabled: "<True|False>"
    weight: "<NN>"
    name: "<Name>"
    block: |
      <Block of configuration. Can use Jinja Templates.>
```

Each file is named like `<weight>-<name>.conf`. Apache will load the files in order according to their weight. Lower numbers load first.

You may use Jinja conditionals in the `enabled` value to enable or disable a block based on OS or some other condition.

Do not forget to add the appropriate `LoadModule` lines. Note that `<aspects_apache24_configuration_root>/modules` is a symlink to the `aspects_apache24_modules_root` directory. So you can load a module like this: `LoadModule modules/modulename.so`.

[defaults/main.yml](defaults/main.yml) contains the bare minimum configuration that worked on the Vagrant test boxes I used. Be sure to review it to make sure it works for your systems.

### `aspects_apache24_configuration_vhosts`
A dictionary of VirtualHost Apache configuration. Each item represents a specific file in the `aspects.vhosts.d` directory. 

Use the following pattern:

```yaml
aspects_apache24_configuration_vhosts:
  <key>:
    enabled: "<True|False>"
    weight: "<NN>"
    name: "<Name>"
    block: |
      <Block of configuration. Can use Jinja Templates.>
```

Each file is named like `<weight>-<name>.conf`. Apache will load the files in order according to their weight. Lower numbers load first.

You may use Jinja conditionals in the `enabled` value to enable or disable a block based on OS or some other condition.

Do not forget to configure `Listen <Port Number>` lines appropriate for your systems. Either in the VirtualHost files, or in the general configuration files. `aspects_apache24_configuration_modules.httpport` in [defaults/main.yml](defaults/main.yml) is configured to tell Apache to listen on port 80.

`aspects_apache24_configuration_vhosts.default` in [defaults/main.yml](defaults/main.yml) is a disabled VirtualHost you may use as an example.

## Configuring Log Files
`<aspects_apache24_configuration_root>/logs` is a symlink to the `aspects_apache24_logs_root` directory. That enables you to reference log file with a path relative to the Apache root configuration directory. Something like:

```yaml
aspects_apache24_configuration_general:
  generallogs:
    enabled: "True"
    weight: "99"
    name: "generallogs"
    block: |
      LogFormat "%h %l %u %t \"%r\" %>s %O" my_custom_format
      {% if ansible_os_family == 'RedHat' %}
      CustomLog "logs/custom.access_log" my_custom_format
      {% elif ansible_os_family == 'Debian' %}
      CustomLog "logs/custom.access.log" my_custom_format
      {% endif %}"
```

`aspects_apache24_configuration_general.generallogformats`, `aspects_apache24_configuration_general.errorlogging`, and `aspects_apache24_configuration_general.defaultaccesslog` in [defaults/main.yml](defaults/main.yml) are configured to match the default settings from RedHat and Debian.

## Overriding configuration
Because `hash_behaviour` is set to `merge` in your `ansible.cfg` file, you can override any dictionary value simply by specifying it's key.

For example:

File: `group_vars/apache.yml`
```yaml
aspects_apache24_configuration_general:
  generallogs:
    enabled: "True"
    weight: "99"
    name: "generallogs"
    block: |
      LogFormat "%h %l %u %t \"%r\" %>s %O" my_custom_format
      {% if ansible_os_family == 'RedHat' %}
      CustomLog "logs/custom.access_log" my_custom_format
      {% elif ansible_os_family == 'Debian' %}
      CustomLog "logs/custom.access.log" my_custom_format
      {% endif %}"
```

File: `host_vars/node1.yml`
```yaml
aspects_apache24_configuration_general:
  generallogs:
    name: "iamawesome"
```

File: `host_vars/node2.yml`
```yaml
aspects_apache24_configuration_general:
  generallogs:
    enabled: "False"
```

Would result in any node in the apache group using the `99-generallogs.conf` file. Except that `node1` would name the file `99-iamawesome.conf`, and `node2` would not have the file at all.

## Deprecated variables
Previous version of the role attempted to coexist with the defaults each OS set. It created a single configuration file that contained all role specific configuration. That method has been superseded. The following variables no longer have any effect.

### `aspects_apache24_default_vhosts` (deprecated)
Dictionary of default virtual host configurations. Used to disable or enable the default vhosts on Ubuntu.

### `aspects_apache24_mods` (deprecated)
Dictionary of modules that you wish to enable or disable. Remember to ensure the correct package is installed before you try to use them.

### `aspects_apache24_httpdconf` (deprecated)
Blocks of apache configuration. Use this to modify the defaults set by the distribution. These are sorted by key. Override blocks by setting the key value to ```""```.

### `aspects_apache24_vhosts` (deprecated)
Blocks of apache vhost configuration. Use this to add new vhosts. These are sorted by key. Override blocks by setting the key value to ```""```.

### `aspects_apache24_other_vhosts_log_enable` (deprecated)
On Debian family systems, enable or disable the other_vhosts.log file. Default is True for enabled. Set to False for disabled. Needs a manual apache restart to apply changes.

## Dependencies
### aspects_packages
[aspects_packages](https://github.com/LaneCommunityCollege/aspects_packages) is used to manage system packages.

## Example Playbook 
```yaml
    - hosts:
      - foo.bar.com
      roles:
      - aspects_apache24
      vars:
        aspects_packages_enabled: True
        aspects_apache24_enabled: True
        aspects_packages_packages:
          modauthcas:
            state: "latest"
            Ubuntu:
              1404: "libapache2-mod-auth-cas"
              1604: "libapache2-mod-auth-cas"
            CentOS:
              7: "mod_auth_cas"
            Debian:
              9: "libapache2-mod-auth-cas"
        aspects_apache24_configuration_modules:
          modhttpproxy:
            enabled: "True"
            name: "auth_proxy"
            weight: "20"
            block: |
              LoadModule proxy_module modules/mod_proxy.so
              LoadModule proxy_http_module modules/mod_proxy_http.so
              <IfModule mod_proxy.c>
              	# If you want to use apache2 as a forward proxy, uncomment the
              	# 'ProxyRequests On' line and the <Proxy *> block below.
              	# WARNING: Be careful to restrict access inside the <Proxy *> block.
              	# Open proxy servers are dangerous both to your network and to the
              	# Internet at large.
              	#
              	# If you only want to use apache2 as a reverse proxy/gateway in
              	# front of some web application server, you DON'T need
              	# 'ProxyRequests On'.
              	#ProxyRequests On
              	#<Proxy *>
              	#   AddDefaultCharset off
              	#   Require all denied
              	#   #Require local
              	#</Proxy>
              	# Enable/disable the handling of HTTP/1.1 "Via:" headers.
              	# ("Full" adds the server version; "Block" removes all outgoing Via: headers)
              	# Set to one of: Off | On | Full | Block
              	#ProxyVia Off
              </IfModule>
        aspects_apache24_configuration_general:
          defaultsslsettings:
            enabled: "True"
            name: "sslsettings"
            weight: "99"
            block: |
              LoadModule ssl_module modules/mod_ssl.so
              # CipherSuite set from https://community.qualys.com/blogs/securitylabs/2013/08/05/configuring-apache-nginx-and-openssl-for-forward-secrecy
              <IfModule mod_ssl.c>
              SSLProtocol All -SSLv2 -SSLv3 -TLSv1
              SSLHonorCipherOrder On
              SSLCipherSuite EECDH+ECDSA+AESGCM:EECDH+aRSA+AESGCM:EECDH+ECDSA+SHA384:EECDH+ECDSA+SHA256:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH+aRSA+RC4:EECDH:EDH+aRSA:RC4:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS:+RC4:RC4
              </IfModule>
        aspects_apache24_configuration_vhosts:
          http_boringdotcom:
            enabled: "True"
            name: "http_boring"
            weight: "20" 
            block: |
              <VirtualHost *:80>
              ServerAdmin webmaster@boring.com
              ServerName boring.com
              ServerAlias www.boring.com
              DocumentRoot /var/www/boring.com
              </VirtualHost>
          duplicatiproxy: 
            enabled: "True"
            name: "http_duplicati"
            weight: "20" 
            block: |
              Listen 8201
              <VirtualHost *:8201>
              ServerAdmin email@example.org
              ServerName {{ ansible_fqdn }}
              #Duplicati Redirect
              AllowEncodedSlashes On
              ProxyPass "/" "http://localhost:8200/"
              ProxyPassReverse "/" "http://localhost:8200/"
              </VirtualHost>
        aspects_apache24_selinux_change: True
        aspects_apache24_selinux_httpd_can_network_connect: False
        aspects_apache24_listen_ports:
          - 8201
```

## License

MIT
