### PAM SSH OIDC

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

