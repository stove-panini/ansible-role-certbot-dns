certbot-dns
=========
Installs Certbot and requests a certificate using a DNS challenge.

Requirements
------------
If using this with RHEL-derivatives, make sure EPEL is installed.

Role Variables
--------------
Names in bold are required.

| Name | Default Value | Comments |
| ---- | ------------- | -------- |
| **certbot_dns_email** | `""` | Email address for important account notifications |
| **certbot_dns_provider** | `""` | DNS provider name as used with the --dns-\<provider\> option |
| **certbot_dns_credentials** | `{}` | Contents of your DNS provider's credentials file |
| certbot\_dns\_domains | `[{{ ansible_fqdn }}]` | List of domains to be used as subjects for the certificate |
| certbot\_dns\_configuration | `{}` | Options as passed to the certbot binary. |
| certbot\_dns\_credentials\_file | `/etc/letsencrypt/<provider>.{ini,json}` | Path to your DNS provider's credentials file |
| certbot\_dns\_force\_renewal | `false` | Whether to renew an existing certificate now, regardless of whether it is near expiry |
| certbot\_dns\_add\_script | `false` | Whether to add the certbot-copy-files script to /usr/local/bin |

The **certbot_dns_configuration** variable is a dictionary whose keys are the same as the command-line options passed to the certbot binary. If the option contains a hyphen, convert it to an underscore (e.g. "--post-hook" becomes "post\_hook"). If an option does not take an argument, like --staging, then its value should be YAML's Null value, "~". See the examples below.

For convenience and to simplify templating the certbot command, **certbot_dns_provider**, **certbot_email**, **certbot_dns_credentials**, **certbot_dns_credentials_file**, and **certbot_dns_domains** have been given their own top-level variables.

The value of **certbot_dns_credentials** should be the key-value pairs that are used to configure your DNS providers' credentials file. Note that Route53 does not use a credentials file. Find documentation for each DNS provider [here](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins).

Example Playbooks
-----------------
These examples have sensitive information in plain text. This is just for demonstration only! You will want to ensure this data is encrypted with Ansible Vault or by some other means.

Minimal example
``` yaml
- hosts: servers
  roles:
    - role: certbot-dns
      vars:
        certbot_dns_email: stephen@trashnet.xyz
        certbot_dns_provider: cloudflare
        certbot_dns_credentials:
          dns_cloudflare_api_token: 11111111111111111111
```

A more featureful example using the LetsEncrypt staging server and hooks
``` yaml
- hosts: servers
  roles:
    - role: certbot-dns
      vars:
        certbot_dns_domains:
          - foo1.example.com
          - foo2.example.com
          - bar.example.com
        certbot_dns_email: stephen@trashnet.xyz
        certbot_dns_provider: google
        certbot_dns_credentials:
          type: service_account
          project_id: my-gcp-project-12345
          private_key_id: asdf1234qwer
          private_key: "-----BEGIN-PRIVATE-KEY-----..."
          client_email: 703723656591-compute@developer.gserviceaccount.com
          client_id: uiop0987hjkl
          auth_uri: https://accounts.google.com/o/oauth2/auth
          token_uri: https://accounts.google.com/o/oauth2/token
          auth_provider_x509_cert_url: https://www.googleapis.com/oauth2/v1/certs
          client_x509_cert_url: https://www.googleapis.com/robot/v1/metadata/x509/703723656591-compute@developer.gserviceaccount.com
        certbot_dns_configuration:
          staging: ~
          post_hook: systemctl restart tomcat
```

I've also included a small program that will copy your certs to a custom directory. You can use this with a deploy hook. See [files/certbot-copy-files](blob/master/files/certbot-copy-files) for usage details.
``` yaml
- hosts: servers
  roles:
    - role: certbot-dns
      vars:
        certbot_dns_domains:
          - foo.example.com
        certbot_dns_email: stephen@trashnet.xyz
        certbot_dns_provider: digitalocean
        certbot_dns_credentials:
          dns_digitalocean_token: 1234567890abcdef
        certbot_dns_add_copy_files_script: true
        certbot_dns_configuration:
          deploy_hook: >
            /usr/local/bin/certbot-copy-files -c /opt/app/foo.crt -k /opt/app/foo.key foo.example.com
```
