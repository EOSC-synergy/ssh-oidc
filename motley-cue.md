# Mapper (`motley-cue`)

Depending on your distribution, install with either:
- `apt-get install motley-cue`
- `yum install motley-cue`
- `zypper install motley-cue`

This will pull a couple of dependencies, most notably `nginx` (for which
epel-release is required on centos7). You SHOULD
get a host-certificate for your server and enable https in nginx. This is
not part of this documentation. 

## Host Configuration

- Open your firewall on port 8080
- **OR** install a hostcertificate with `nginx` and open your firewall on
    port 443

- Notes on selinux (for centos7 and centos8):
    - We tested `pam-ssh-oidc` and `motley-cue` in permissive mode only.
    - We succeeded in running in enforced mode, by using these commands:
        ```config
        semodule -i /usr/share/motley-cue/selinux/motley-cue-gunicorn.pp
        semodule -i /usr/share/motley-cue/selinux/motley-cue-sshd.pp
        semodule -i /usr/share/motley-cue/selinux/motley-cue-nginx.pp
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
You can support multiple OPs and configure authorisation for each OP separately.
There are three options to authorise users:
- authorise all: allow all users from a trusted OP
- individual: authorise single users via their unique identifier given by `sub` + `iss`
- VO-based: authorise users that are members of a specific VO (or a set of VOs)

The VO definitions allowed include:
- String list of groups
- Entitlements according to [AARC-G002 Expressing group membership and role information](https://aarc-community.org/wp-content/uploads/2017/11/AARC-JRA1.4A-201710.pdf)
where the claim containing the VOs is configurable as well.

The default configuration contains several examples, but you'll need to modify `/etc/motley_cue/motley_cue.conf` to enable any authorisation.

You can find out your own groups with:
```
pip install flaat
flaat-userinfo --oidc egi
```



