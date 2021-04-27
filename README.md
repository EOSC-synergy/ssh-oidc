# What is pam-ssh-oidc

It's a set of tools that allows (you guessed it) ssh with OIDC. What you
might not have guessed: We go for the difficult-to-implement but
simplest-to-use approach.

## Usability

- No modification of ssh-client (except for Windows where we provide an extenstion to putty)
- No modification of ssh-server
- No need for OIDC client registration on the server
- No need to enter passwords more than once after reboot

## Features
The tools the we use and describe in this context will offer many
features.

- The PAM MODULE **pam-ssh-oidc** allows ssh to accept Access Tokens in
    addition to passwords.
- The Mapping Daemon **motley-cue** is an additional daemon on the ssh
    server. It is in charge of "the magic behind the scenes" and deserves
    special attention by the ssh server administrators! It is in charge of
    several tasks:
    - Authorisation Decision: Is an incoming user welcome on the system or
        not. `motley-cue` offers **two dimensions** for making this decision:
        - **Membership**: 
            - Memebership in a *Virtual Organisation* (a
                group managed remotely, e.g. on an OIDC Provider). A system
                administrator essentially delegates the decision which users
                may log in to the system to the administrator of the Virtual
                Organisation.
            - *Individual Users*: Of course, it is also possible to authorise
                individual users (just as before). However, now users are
                identified by the OIDC `sub` and `iss` claims.
        - **Assuranc**: The second dimension is based on **how well a user
            is known**. We use the [Refeds Assurance
            Framework](https://refeds.org/assurance) to describe a user.
            
        This allows to only give logins to users that are in a
        specific group AND have a certain level of assurance.
- A commandline tool **oidc-agent** that provides the functions for
`ssh-agent` but for ssh.
- A commandline wrapper **mcc-ssh** that 
    - gets you an AccessToken
    - gets you an account on the remote (talks with `motley-cue`)
    - calls ssh and passes the AccessToken

# Installation

Packages are available at [https://repo.data.kit.edu](https://repo.data.kit.edu)

Follow the instructions there to support the correct repository for apt or
yum.

## Server
The currently supported Linuxes are:
- Debian (stable + testing)
- Ubuntu (20.04 + 18.04)
- Centos 8

### PAM Module

Install with either
- `apt-get install pam-ssh-oidc`
- `yum install pam-ssh-oidc`

The installation will modify `/etc/pam.d/sshd` to add support for
pam-ssh-oidc. The pam module is configured in `/etc/pam.d/pam-ssh-oidc-config.ini`

You can verify it's working by `ssh <anything>@<hostname you installed on>`
You should be prompted with
```
Access Token: 
```
If you do not see this prompt, please verify that you have enabled this in
`/etc/ssh/sshd_config`:
```
ChallengeResponseAuthentication yes
```

If you still do not see the prompt, please [open an issue](https://github.com/EOSC-synergy/pam-ssh-oidc/issues)

### Mapper (`motley-cue`)

Install with either
- `apt-get install motley-cue`
- `yum install motley-cue`

This will pull a couple of dependencies, most notably `nginx`. You SHOULD
get a host-certificate for your server and enable https in nginx. This is
not part of this documentation. 

Configuration of `motley-cue` takes place in `/etc/motley-cue`:
- `gunicorn.conf.py`: Daemon configuration (glues it toegether with nginx (via `/etc/nginx/sites-enabled/nginx.motley_cue`)
- **`motley_cue.conf`: Authorisation configuration**: Allows definition of
    the Virtual Organisation (or groups) that are allowed to use the service.
- **`feudal_adapter.conf`: Account creation configuration**: Allows (among
    others to configure the account creation, including the levels of
    assurance. 
- `motley_cue.env`: Environment variables for `motley_cue`

#### Assurance Configuration
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

#### Authorisation Configuration
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

## Client Installation

On the client you will need two basic tools:

- oidc-agent: To obtain oidc AccessTokens
- motley-cue shell (`mc_ssh`) for 
    - getting AccessTokens
    - communicating with the remote motley-cue
    - Calling SSH with an AccessToken


### `oidc-agent`
oidc agent is available as packages via [https://repo.data.kit.edu](https://repo.data.kit.edu)

Install with either of:
- `apt-get install oidc-agent`
<!--FIXME-->
- add commandline for archlinux here
- `yum install oidc-agent`
- `brew install oidc-agent`
    
You will need to create an oidc-agent configuration. The shortest commandline for this is:
` oidc-gen --pub --issuer https://aai.egi.eu/oidc --scope "email eduperson_assurance offline_access" egi`

For more information there is a [gitbook](https://indigo-dc.gitbooks.io/oidc-agent) and the
(gitlab)[https://github.com/indigo-dc/oidc-agent] page.

### `mc_ssh`

Install with

- `pip install mc-ssh`

Use with 

`mccli ssh <hostname>`

It is as simple as this!

# More Material

We have two presentations:
- The (short) overview: [https://docs.google.com/presentation/d/18GVVwuf3Ham0PBdnVf2MJm96PUPGBU_zzglfskG9LtY](https://docs.google.com/presentation/d/18GVVwuf3Ham0PBdnVf2MJm96PUPGBU_zzglfskG9LtY)
- The detailed one: [https://docs.google.com/presentation/d/1CExFLow9ZRQtU4vVSnBee9Cq8_2UAPAgr7uZpjubTWY](https://docs.google.com/presentation/d/1CExFLow9ZRQtU4vVSnBee9Cq8_2UAPAgr7uZpjubTWY)

# Acknowledgements

This page documents a set of tools that have been developed in a joint
effort of:
- Karlsruhe Institute of Technology (KIT)
- Poznan Supercomputing and Networking Centre (PSNC)
- EOSC-Synergy
- Praceclab PL
- Helmholtz Federated IT Services (HIFIS)
- Helmholtz Data Federation
