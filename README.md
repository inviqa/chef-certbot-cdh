# certbot-cdh

The certbot-cdh integrates certbot with config-driven-helper sites, to automatically
set up and link the SSL certificates to each site.

It by default creates a single certificate shared between each site, adding all
['server_name']s and ['server_aliases'] to the certificate.

It can optionally split up the certificates into separate sites based on site
['ssl']['use_sni'] and ['ssl']['san_group'] settings.

Usage
-----

Add `"recipe[certbot-cdh]"` to enable it.

Include the following in attributes:

```json
"default_attributes": {
  "certbot": {
    "cert-owner": {
      "email": "devops@inviqa.com"
    }
  },
  "nginx": {
    "shared_config": {
      "<project-name>": {
        "protocols": ["http", "https"],
        "includes_first": [
          "certbot.conf"
        ]
      }
    }
  }
}
```

Add the following cookbooks to the Berksfile:

```text
cookbook 'config-driven-helper', '~> 2.5'
cookbook 'certbot-cdh', '~> 0.1.0'
```

Given you have nginx or apache sites defined for example as:


```json
"default_attributes": {
  "nginx": {
    "sites": {
      "mysite1": {
        "server_name": "mysite1.dev",
        "docroot": "/var/www/mysite1/public",
        "inherits": "<project name>"
      },
      "mysite2": {
        "server_name": "mysite2.dev",
        "server_aliases": ['static.mysite2.dev'],
        "docroot": "/var/www/mysite1/public",
        "inherits": "<project name>"
      },
    }
  }
}
```

This will create letsencrypt cert/chain/fullchain/privkey pem files in:

/etc/letsencrypt/live/mysite1.dev/

The certificate will have SAN domains:
mysite1.dev
mysite2.dev
static.mysite2.dev

Certbot uses the first domain of the certificate's domains as the folder to store
them in.

Node attributes for the sites will automatically be set up to point ['ssl']['certfile'],
['ssl']['certchainfile'], and ['ssl]['keyfile'] to the correct pem files for
each site.

Where apache will use:
['ssl']['certfile'] = /etc/letsencrypt/live/mysite1.dev/cert.pem
['ssl']['certchainfile'] = /etc/letsencrypt/live/mysite1.dev/chain.pem
['ssl']['keyfile'] = /etc/letsencrypt/live/mysite1.dev/privkey.pem

And nginx will use:
['ssl']['certfile'] = /etc/letsencrypt/live/mysite1.dev/fullchain.pem
['ssl']['keyfile'] = /etc/letsencrypt/live/mysite1.dev/privkey.pem

config-driven-helper::apache-sites and config-driven-helper::nginx-sites will
use this to set up their vhost's ssl configuration.

See the spec for examples of using ['ssl']['use_sni'] and ['ssl']['san_group']
to split up the certificates per config-driven-helper site.

License and Authors
-------------------
- Author:: Andy Thompson
- Author:: Felicity Ratcliffe

```text
Copyright:: 2016 The Inviqa Group Ltd

See LICENSE file
```
