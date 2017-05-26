# dovecot

#### Table of Contents

1. [Description](#description)
1. [Setup - The basics of getting started with dovecot](#setup)
    * [What dovecot affects](#what-dovecot-affects)
    * [Beginning with dovecot](#beginning-with-dovecot)
1. [Usage - Configuration options and additional functionality](#usage)
1. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
1. [Limitations - OS compatibility, etc.](#limitations)
1. [Development - Guide for contributing to the module](#development)

## Description

This module installs and manages the dovecot imap server and its plugins, and provides
resources and functions to configure the dovecot system. 
It does, however, not configure any of those systems beyond the upstream defaults.

This module is intended to work with Puppet 4 and has been tested with 
dovecot 2.2.22 on Ubuntu 16.04. Patches to support other setups are welcome.

## Setup

### What dovecot affects

By default, this package...

* installs the dovecot package
* recursively purges all dovecot config

### Beginning with dovecot

While on a puppet-managed host, splitting the config into multiple conf.d files provides 
not much advantage, this module supports managing both the dovecot.conf file and several
conf.d files, by specifying the file param.

The dovecot class takes two parameters, $config for dovecot.conf entries and $configs for
conf.d file entries:

```puppet
class { 'dovecot':
  plugins => ['imap', 'lmtp'],
  config => {
    protocols => 'imap lmtp',
    listen    => '*, ::',
  },
  configs => {
    '10-auth' => {
      passdb => {
        driver => 'passwd-file',
        args   => 'username_format=%u /etc/dovecot/virtual_accounts_passwd',
      },
    },
    '10-logging' => {
      log_path => 'syslog',
    },
  }
}
```

This can be conveniently used from hiera:

```yaml
dovecot::plugins:
  - imap
  - lmtp
  - sieve
dovecot::config:
  protocols: imap sieve lmtp
  hostname: "%{::fqdn}"  
dovecot::configs:
  '10-auth':
    disable_plaintext_auth: yes
    passdb:
      driver: passwd-file
      args: scheme=CRYPT username_format=%u /etc/dovecot/virtual_accounts_passwd
  '10-master':
    default_process_limit: 200
    default_client_limit: 2000
    service lmtp:
      unix_listener /var/spool/postfix/private/dovecot-lmtp:
        user: postfix
        group: postfix
        mode: '0600'
  '10-ssl':
    ssl: yes
    ssl_cert: '</etc/dovecot/ssl/dovecot.crt'
    ssl_key: '</etc/dovecot/ssl/dovecot.key'    
```



If you want to use the dovecot:config resource directly, the easiest way is to put both the 
file (optional) and the hierachical config key into the resource title:

```puppet
dovecot::config {
  'protocols': value => 'imap lmtp';
  'listen': 
     value => '*, ::',
     comment => 'Listen on all interfaces',
  ;
  '10-auth:passdb.driver': value => 'passwd-file';
  '10-auth:passdb.args': value => 'username_format=%u /etc/dovecot/virtual_accounts_passwd'
}
```

For advanced use-cases you can also use the provided `dovecot::create_config_resources` and 
`dovecot::create_config_file_resources` functions, that are used to handle the $config and 
$configs parameters.



## Reference

See the reference generated by puppet strings on https://oxc.github.io/puppet-dovecot/

## Limitations

OS Versions tested:

* Ubuntu 16.04

dovecot versions tested:

* 2.2.22

Feel free to let me know if it correctly works on a different OS/setup, or 
submit patches if it doesn't.

## Development

You're welcome to submit patches and issues to the issue tracker on Github.

