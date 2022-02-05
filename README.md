certbot-dns
=========
Installs Certbot and requests a certificate using a DNS challenge.

Requirements
------------
If using this with RHEL-derivatives, make sure EPEL is installed.

Role Variables
--------------
| Name | Default Value | Comments |
| ---- | ------------- | -------- |
| certbot\_dns\_email | `me@example.com` | Email address for important account notifications |
| certbot\_dns\_domains | `[{{ ansible_fqdn }}]` | List of domains to be used as subjects for the certificate |
| certbot\_dns\_propagation\_seconds | `10` | The number of seconds to wait for DNS to propagate before asking the ACME server to verify the DNS record |
| certbot\_dns\_acme\_server | `https://acme-v02.api.letsencrypt.org/directory` | ACME directory resource URI. Defaults to LetsEncrypt. |
| certbot\_dns\_provider | `cloudflare` | DNS provider name as given to the --dns-\<provider\> option |
| certbot\_dns\_credentials\_file | `/etc/letsencrypt/{{ certbot_dns_provider }}.ini` | INI file containing credentials for your DNS provider |
| certbot\_dns\_credentials | `{dns_cloudflare_email: webadmin@example.com, dns_cloudflare_api_key: abcd-1234}` | INI file contents

The value of certbot\_dns\_credentials should be the key-value pairs that are used to configure your DNS providers' credentials file. See https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins

Example Playbook
----------------
``` yaml
- hosts: servers
  roles:
    - role: certbot-dns
      vars:
        certbot_dns_email: stephen@trashnet.xyz
        certbot_dns_credentials:
          dns_cloudflare_api_token: 11111111111111111111
```
