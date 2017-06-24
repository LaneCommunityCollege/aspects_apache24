# aspects_apache24
Part of the Aspects Suite.

Allows you to configure Apache 2.4 server and virtual hosts.

## Requirements

Set ```hash_behaviour=merge``` in your ansible.cfg file.

## Role Variables
See the files in defaults for examples of most variables. If any are not self explanatory, see the Apache 2.4 docs. Each individual configuration option is named the same, or similar, to the Apache directive it controls.

* ```aspects_apache24_enabled```: Flag to enable or disable this role. Default is disabled.
* ```aspects_apache24_packages```: The various Apache 2.4 packages you can install.
* ```aspects_apache24_default_vhosts```: Dictionary of default virtual host configurations. Used to disable or enable the default vhosts on Ubuntu.
* ```aspects_apache24_mods```: Dictionary of modules that you wish to enable or disable. Remember to ensure the correct package is installed before you try to use them.
* ```aspects_apache24_httpdconf```: Blocks of apache configuration. Use this to modify the defaults set by the distribution. These are sorted by key. Override blocks by setting the key value to ```""```.
* ```aspects_apache24_vhosts```: Blocks of apache vhost configuration. Use this to add new vhosts. These are sorted by key. Override blocks by setting the key value to ```""```.

## Example Playbook
```yaml
    - hosts:
      - foo.bar.com
      roles:
      - aspects_apache24
      vars:
        aspects_apache24_enabled: True
        aspects_apache24_packages:
          modauthmysql:
            state: "latest"
        aspects_apache24_mods:
          modauthmysql:
            Debian:
              enable: True
              links:
                loadfile: "auth_mysql.load"
        aspects_apache24_httpdconf:
          1defaultssl: |
            # 1defaultssl config from aspects_apache24/defaults/main.yml
            # CipherSuite set from https://community.qualys.com/blogs/securitylabs/2013/08/05/configuring-apache-nginx-and-openssl-for-forward-secrecy
            <IfModule mod_ssl.c>
              SSLProtocol All -SSLv2 -SSLv3 -TLSv1
              SSLHonorCipherOrder On
              SSLCipherSuite EECDH+ECDSA+AESGCM:EECDH+aRSA+AESGCM:EECDH+ECDSA+SHA384:EECDH+ECDSA+SHA256:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH+aRSA+RC4:EECDH:EDH+aRSA:RC4:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS:+RC4:RC4
            </IfModule>
        aspects_apache24_vhosts:
          2boringdotcom: |
            <VirtualHost *:80>
              ServerAdmin webmaster@boring.com
              ServerName boring.com
              ServerAlias www.boring.com
              DocumentRoot /var/www/boring.com
              AuthName "MySQL Testing"
              AuthType Basic
              AuthMySQLHost localhost
              AuthMySQLDB test
              AuthMySQLUserTable user_info
              AuthMySQLEnable On
              require valid-user
            </VirtualHost>
```
## How it works
Default configuration is modified so that all the custom configuration in the ```zzzzzCustomHttpd.conf``` file will override whatever it needs to override.

On Debian/Ubuntu, that file is placed at: ```/etc/apache2/conf-enabled/zzzzzCustomHttpd.conf```

## Installing Modules
* Find the correct module in ```aspects_apache24_packages```
* Set ```state``` to ```latest``` or ```present```.

```yaml
    aspects_apache24_packages:
      modulekey:
        state: "latest"
```

If the module is not in ```aspects_apache24_packages``` already, or doesn't have a package name for your OS, just do something like:

```yaml
    aspects_apache24_packages:
      modulekey:
        state: "latest"
        OS: "package name"
```

## Overriding values
If you have a ```aspects_apache24_httpdconf``` section set in a group_vars file but you want to get rid of it, or modify it on an individual server, just find the key, and set in your host_vars file. That's why we use ```hash_behaviour=merge```.

## License

MIT
