# What is ssh-oidc

It's a set of tools that allows (you guessed it) ssh with OIDC. What you
might not have guessed: We go for the difficult-to-implement but
simplest-to-use approach.

## Usability

- No modification of ssh-client (except for Windows where we provide an extenstion to putty)
- No modification of ssh-server
- No need for OIDC client registration on the server
- No need to enter passwords more than once after reboot

The final usage is as simple as:
```
mccli ssh <hostname>
```

<!--## Features-->
<!--The tools the we use and describe in this context will offer many-->
<!--features.-->
<!---->
<!--- The PAM MODULE **pam-ssh-oidc** allows ssh to accept Access Tokens in-->
<!--    addition to passwords.-->
<!--- The Mapping Daemon **motley-cue** is an additional daemon on the ssh-->
<!--    server. It is in charge of "the magic behind the scenes" and deserves-->
<!--    special attention by the ssh server administrators! It is in charge of-->
<!--    several tasks:-->
<!--    - Authorisation Decision: Is an incoming user welcome on the system or-->
<!--        not. `motley-cue` offers **two dimensions** for making this decision:-->
<!--        - **Membership**: -->
<!--            - Memebership in a *Virtual Organisation* (a-->
<!--                group managed remotely, e.g. on an OIDC Provider). A system-->
<!--                administrator essentially delegates the decision which users-->
<!--                may log in to the system to the administrator of the Virtual-->
<!--                Organisation.-->
<!--            - *Individual Users*: Of course, it is also possible to authorise-->
<!--                individual users (just as before). However, now users are-->
<!--                identified by the OIDC `sub` and `iss` claims.-->
<!--        - **Assuranc**: The second dimension is based on **how well a user-->
<!--            is known**. We use the [Refeds Assurance-->
<!--            Framework](https://refeds.org/assurance) to describe a user.-->
<!--            -->
<!--        This allows to only give logins to users that are in a-->
<!--        specific group AND have a certain level of assurance.-->
<!--- A commandline tool **oidc-agent** that provides the functions for-->
<!--`ssh-agent` but for ssh.-->
<!--- A commandline wrapper **mcc-ssh** that -->
<!--    - gets you an AccessToken-->
<!--    - gets you an account on the remote (talks with `motley-cue`)-->
<!--    - calls ssh and passes the AccessToken-->



# Client Installation

**For testing the client, we provide a demonstration server at
[ssh-oidc-demo](https://ssh-oidc-demo.data.kit.edu) server**


On the client you will need two basic tools:

- oidc-agent: To obtain oidc AccessTokens
- motley-cue shell (`mc_ssh`) for 
    - getting AccessTokens
    - communicating with the remote motley-cue
    - Calling SSH with an AccessToken


## oidc-agent
Please follow installation instructions at
[https://indigo-dc.gitbook.io/oidc-agent/installation](https://indigo-dc.gitbook.io/oidc-agent/installation)

Useful commandlines for generating an oidc-agent configuration are:
- **EGI Check-in**:
    ```
    oidc-gen --pub --iss https://aai.egi.eu/oidc \
        --scope "openid profile email offline_access \
            eduperson_entitlement eduperson_scoped_affiliation eduperson_unique_id" egi
    ```
- **WLCG**: 
    ```
    oidc-gen --pub  --issuer https://wlcg.cloud.cnaf.infn.it/ \
        --scope "openid profile offline_access wlcg.groups wlcg \
            eduperson_entitlement eduperson_scoped_affiliation " wlcg
    ```
- **Helmholtz-AAI**: 
    ```
    oidc-gen --pub --iss https://login.helmholtz.de/oauth2/ \
        --scope "openid profile email offline_access \
            eduperson_entitlement eduperson_scoped_affiliation eduperson_unique_id" helmholtz
    ```
- **Google**: 
    ```
    oidc-gen --pub --iss https://accounts.google.com/ --flow device  --scope max google
    ```

<!--
oidc agent is available as packages via [https://repo.data.kit.edu](https://repo.data.kit.edu)

Follow the instructions there to support the correct repository for apt or yum.

The currently supported Linuxes are:
- Debian (stable + testing)
- Ubuntu (20.04 + 18.04)
- Centos (7 + 8)

Install with either of:
- `apt-get install oidc-agent`
- `yum install oidc-agent`

For any other Linux distributions, you will have to [install from source](https://indigo-dc.gitbook.io/oidc-agent/installation/install#from-source).

For MacOS, follow the instructions at [https://indigo-dc.gitbook.io/oidc-agent/macos/installation](https://indigo-dc.gitbook.io/oidc-agent/macos/installation).
    
You will need to create an oidc-agent configuration. The shortest commandline for this is:
` oidc-gen --pub --issuer https://aai.egi.eu/oidc --scope "openid profile email offline_access eduperson_entitlement eduperson_scoped_affiliation eduperson_unique_id" egi`

For more information there is a [gitbook](https://indigo-dc.gitbooks.io/oidc-agent) and the
[github](https://github.com/indigo-dc/oidc-agent) page.
-->

## mc_ssh

Install with

- `pip install mc-ssh`

Use either of:

- `mccli ssh ssh-oidc-demo.data.kit.edu --iss https://aai.egi.eu/oidc`
- `mccli ssh ssh-oidc-demo.data.kit.edu --oidc egi`
- `ACCESS_TOKEN=<access_token> mccli ssh ssh-oidc-demo.data.kit.edu`

It is as simple as this!


# Server Installation

Packages are available at [https://repo.data.kit.edu](https://repo.data.kit.edu)

Follow the instructions there to support the correct repository for apt or yum.

The currently supported Linuxes are:
- Debian (stable + testing)
- Ubuntu (20.04 + 18.04)
- Centos (7 + 8)

Installation is mostly a matter of installing the packages:
- `motley-cue` and `pam-ssh-oidc`

Details are described in the linked chapters
[pam-ssh-oidc](pam-ssh-oidc.md) and [motley-cue](motley-cue.md)


# More Material

We have two presentations:
- The (short) overview: [https://docs.google.com/presentation/d/18GVVwuf3Ham0PBdnVf2MJm96PUPGBU_zzglfskG9LtY](https://docs.google.com/presentation/d/18GVVwuf3Ham0PBdnVf2MJm96PUPGBU_zzglfskG9LtY)
- The detailed one: [https://docs.google.com/presentation/d/17HM11YjafC5VA4_o2EjNrtbRqJGgQP0q92C_uqFAM6A/edit?usp=sharing](https://docs.google.com/presentation/d/17HM11YjafC5VA4_o2EjNrtbRqJGgQP0q92C_uqFAM6A/edit?usp=sharing)

# Acknowledgements

This page documents a set of tools that have been developed in a joint
effort of:
- Karlsruhe Institute of Technology (KIT)
- Poznan Supercomputing and Networking Centre (PSNC)
- EOSC-Synergy
- Praceclab PL
- Helmholtz Federated IT Services (HIFIS)
- Helmholtz Data Federation
