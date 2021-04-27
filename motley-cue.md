# Mapper (`motley-cue`)

Install with either
- `apt-get install motley-cue`
- `yum install motley-cue`

This will pull a couple of dependencies, most notably `nginx`. You SHOULD
get a host-certificate for your server and enable https in nginx. This is
not part of this documentation. 

## Host Configuration

- Open your firewall on port 8080
- **OR** install a hostcertificate with `nginx` and open your firewall on
    port 443

- Notes on selinux (for centos7 and centos8):
    - We tested `pam-ssh-oidc` and `motley-cue` in permissive mode only.
    - We succeeded in running in enforced mode on centos8, by using these
        commands:
        ```config
        semodule -X 300 -i my-gunicorn.pp
        semodule -X 300 -i my-sshd.pp
        setsebool -P nis_enabled 1
        ```
    - Feedback is appreciated

## motley-cue configuration

Configuration of `motley-cue` takes place in `/etc/motley-cue`:
- `gunicorn.conf.py`: Daemon configuration (glues it toegether with nginx (via `/etc/nginx/sites-enabled/nginx.motley_cue`)
- **`motley_cue.conf`: Authorisation configuration**: Allows definition of
    the Virtual Organisation (or groups) that are allowed to use the service.
- **`feudal_adapter.conf`: Account creation configuration**: Allows (among
    others to configure the account creation, including the levels of
    assurance. 
- `motley_cue.env`: Environment variables for `motley_cue`

## Assurance Configuration
The config file is self documenting. Even more information
may be found [here](https://git.scc.kit.edu/feudal/feudalAdapterLdf).
By default from the motley-cue package, the following defaults are
set:
- Username creation: 
    - Try to respect incoming `preferred_username`
    - If not iterate over combinations of first and last name
- Assurance:
```
require = profile/espresso |
    IAP/medium & ID/eppn-unique-no-reassign |
    IAP/low & ID/eppn-unique-no-reassign |
    https://aai.egi.eu/LoA#Substantial |
    profile/cappuccino
```
- This allows the following users:
    - users of university IdPs (IAP/medium or profile/cappuccino)
    - users of university IdPs according to an older spec (https://aai.egi.eu/LoA#Substantial)
    - users of social IdPs (IAP/low)
    - And requires a unique, non-rassignable identifier
- Assurance definition is used from the `eduperson_assurance` claim
and follows these specifications:
    - [Refeds Assurance Framework (RAF)](https://refeds.org/assurance)
    - [AARC-G021 Exchange of specific assurance information between Infrastructures](https://aarc-community.org/guidelines/aarc-g021)

## Authorisation Configuration
The group definitions allowed include:
- String list of groups
- Entitlements according to [AARC-G002 Expressing group membership and role information](https://aarc-community.org/wp-content/uploads/2017/11/AARC-JRA1.4A-201710.pdf)
- The default is:
```
group = [
    "urn:geant:helmholtz.de:group:KIT#login-dev.helmholtz.de",
    'urn:mace:egi.eu:group:mteam.data.kit.edu:role=member#aai.egi.eu',
    "urn:mace:egi.eu:group:eosc-synergy.eu"
]
```
- You can find out your own groups with this client comment. (See installation for the client later):

    `flaat-userinfo --oidc egi`


