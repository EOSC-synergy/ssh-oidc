# Frequently Asked Questions

## Client

1. My login does not work / I'm prompted with `Password: `
    - This is probably due to your Access Token being too long.  Access
        Tokens that exceed a length of 1023 characters cannot be processed
        by ssh.  To find out about the length of your Access Token, run:

    `oidc-token | wc -c`

1. `oidc-agent` seems to hang.
    - This is due to a bug in libcjson, and should not appear any longer.
        It was fixed in oidc-agent 4.1.0-4 (and 4.1.0-5 for ubuntu 20). 
    - Contact [oidc-agent-contact@lists.kit.edu](oidc-agent-contact@lists.kit.edu)

1. I cannot run `mccli` after installing it via `pip` (or `pip3`).
    - This may be due to `$HOME/.local/bin` not being in your `$PATH`.
    - Try `export PATH=$PATH:$HOME/.local/bin` to fix it.

1. After running first time I get: 
    `error: Could not infer motley_cue endpoint from command`
    - Try updating mccli with: `pip install -U mccli`
    - Try adding `--mc-endpoint https://<hostname>:8443` to the mccli command
    - Try adding `--mc-endpoint http://<hostname>:8080` to the mccli command
    - Ask a service administrator whether `motley_cue` runs on a non-standard port

1. I get an error that I am not authorised:
    `error: Failed on get_status: [HTTP 403] "Forbidden: you are not authorised to access this service"`
    - Check if your token issuer is supported on the service:
    
        `mccli --oidc <oidc-agent account name> info <hostname>`
    - In the command output, check that the issuer URL appears in the list of `supported OPs`.
    - Look at the information in the section: `Authorisation on service for provided token issuer (OP)`.
        - If the authorisation is VO-based, make sure you are a member of an authorised VO: the section `Information retrieved from userinfo endpoint` in the output of the command above.
        - In the case of individual user authorisation, contact the service administrator. 

## Server

1. No matter which token I use, I get an error that I am not authorised:
    `error: Failed on get_status: [HTTP 403] "Forbidden: you are not authorised to access this service"`
    - no users are authorised by default, you first have to configure the authorisation in `/etc/motley_cue/motley_cue.conf`
    - the config file is quite verbose and lists all the available features with examples; or check out the docs on authorisation [here](motley-cue.md#authorisation-configuration).

1. I configured authorisation for my preferred issuer but I'm still not authorised.
    - make sure that you don't have any leading spaces for any options configured in `motley_cue.conf` (e.g. `authorised_vos`) 
    - the .ini file format used by `motley_cue.conf` interprets leading spaces as a line continuation of the previous option. 

1. Is it mandatory to use host certificates and run `motley-cue` on port 443?
    - no, by default `motley-cue` actually runs on port 8080
    - it is RECOMMENDED to use a host certificate, but `mccli` will work without it as well and only issue a warning that your connection is insecure
    - you can configure `motley_cue` to run on any port you'd like in the nginx configuration:
        - `/etc/nginx/sites-available/nginx.motley_cue` (on debian-based systems) or
        - `/etc/nginx/conf.d/nginx.motley_cue.conf` (on RPM-based systems)
    - the recommended ports are 443 or 8443 for HTTPS and 8080 for HTTP; otherwise, inform the users if you use non-standard ports so that they can pass `--mc-endpoint <motley_cue URL>` to `mccli`.

1. Can I use self-signed certificates?
    - Please don't. Obtaining a certificate via [Let's Encrypt](https://letsencrypt.org/) is really easy to set up.
    - If you still want to do this, for testing purposes, you can tell `mccli` to ignore the certificate verification via `--insecure`.

1. I get this error message: `error: Failed on deploy: [HTTP 403] [state=rejected] Your assurance level is insufficient to access this resource`
    - assurance is controlled in `/etc/motley_cue/feudal_adapter.conf`
    - you can disable it by adding (in the `[assurance]` section): `skip = Yes, do as I say!`
    - more details on how to configure it [here](motley-cue.md#assurance-configuration).
