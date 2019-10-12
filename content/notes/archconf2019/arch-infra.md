---
title: "Arch Infra"
date: 2019-10-12T16:53:24+02:00
---

## done on 2019-10-06

- migrated forums to discourse using a public migration script
- set up keycloak with Terraform automation
- set up GitLab with saml2 

## User management

Explored managing the users using Ansible to manage LDAP. Flaws with LDAP: firstname/lastname should be set which some software does not handle, slow for searching.
KDE is moving to openid-connect. Start with gitlab, and extend from there.

Services:
    - Grafana
    - BBS
    - Zabbix
    - Mediawiki
    - AUR
    - Patchwork (gitlab?)
    - Archweb
    - Mailman
    - Kanboard (gitlab?)
    - Gitlab
    - Matrix
    - Quassel
    - Email (dovecot) => unix user
    - SSH/unix users
    
Databases:
    - bbs:    94000
    - aur:     66531
    - bugs:  31075
    - wiki:    35000

* Questions
- To we put everyone in openid connect or not?
- How to do this gradually?
- How well does it scale?

* Next Steps

- Package freeipa
- Package keycloak
- Getting Arch Users in Keycloak and decide on the attributes
- Make Gitlab use keycloak


## Gitlab

Replace https://git.archlinux.org

* Restrict CI to master branches]
- why?

Steps:
    - reproduce the current repos

## BBS Alternative

- Retain old posts
- GDPR (or make it easier then the curent scenario)
- OAUTH or Plugin software
- Spam / signup protection such as we have now
- Moderation tools, mass removing posts. 
- Theme-ability

Possible alternatives:
    - dfeed https://github.com/CyberShadow/DFeed
    - discourse
     https://meta.discourse.org/t/migrating-to-discourse-from-another-forum-software/16616 
     https://github.com/discourse/discourse/blob/master/script/import_scripts/fluxbb.rb
     http://docs.mailman3.org/en/latest/config-web.html
   - https://flarum.org

## Mailman 2

http://docs.mailman3.org/en/latest/config-web.html#configure-login-to-django
https://docs.mailman3.org/en/latest/migration.html
http://docs.mailman3.org/en/latest/config-web.html


## DNS Providers

Have terraform provide dns


