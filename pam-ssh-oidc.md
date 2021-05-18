### PAM SSH OIDC

We offer two packages for installation: `pam-ssh-oidc` and
`pam-ssh-oidc-autoconfig`. The difference being that the `-autoconfig`
version will automatically patch/create `/etc/pam.d/sshd`, to allow sshd
to prompt for Access Tokens via Pluggable Authentication Module (PAM).


Install with either
- `apt-get install pam-ssh-oidc-autoconfig`
- `yum install pam-ssh-oidc-autoconfig`

or

- `apt-get install pam-ssh-oidc`
- `yum install pam-ssh-oidc`

The pam module itself is configured in `/etc/pam.d/pam-ssh-oidc-config.ini`,
the default pointing to the `motley-cue` instance, which is assumed to be
on localhost.

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

# Manual Configuration

For information on configuration, please 
[see the Readmes of pam-ssh-oidc packaging](https://github.com/EOSC-synergy/pam-ssh-oidc-packaging/blob/master/documentation/README-pam-ssh-oidc.md)
page.

If you still do not see the prompt, please [open an issue](https://github.com/EOSC-synergy/pam-ssh-oidc/issues)

