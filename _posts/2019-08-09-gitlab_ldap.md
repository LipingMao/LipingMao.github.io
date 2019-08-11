---
title: DevOps -- Enable Gitlab LDAP
---

## Background

> Enable ldap for gitlab is required when deploy gitlab in private environment, here is a how-to blog to help you enable ldap.

## Steps

Here is how it looks like in configuration(/etc/gitlab/gitlab.rb):

```
### LDAP Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/ldap.html
###! **Be careful not to break the indentation in the ldap_servers block. It is
###!   in yaml format and the spaces must be retained. Using tabs will not work.**

# gitlab_rails['ldap_enabled'] = false
gitlab_rails['ldap_enabled'] = true

###! **remember to close this block with 'EOS' below**

gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
   main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     host: '10.0.4.89'
     port: 389
     uid: 'uid'
     bind_dn: 'cn=Manager,dc=panorama,dc=com'
     password: 'PASSWORD' # replace your password
     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
     verify_certificates: false
     smartcard_auth: false
     active_directory: false
     allow_username_or_email_login: false
     lowercase_usernames: false
     block_auto_created_users: false
     base: 'ou=Employees,ou=Pano Users,dc=panorama,dc=com'
     user_filter: ''
EOS
```
