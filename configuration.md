## Configuration

Configuration is only required on the server side.
The system is configured via the following configuration files.
(`oidc-agent` is considered out of scope for this documentation.)

### `/etc/pam.d/sshd`

A single line in the beginning of `/etc/pam.d/sshd` is sufficient to
enable `pam-ssh-oidc`:
```config
auth   sufficient pam_oidc_token.so config=/etc/pam.d/pam-ssh-oidc-config.ini
```
This line is automatically added when using the `pam-ssh-oidc-autoconfig`
package.

### `/etc/ssh/sshd_config`

SSHD should be configured to accept `KbdInteractiveAuthentication`
(previously `ChallengeResponseAuthentication`):

```config
ChallengeResponseAuthentication yes
KbdInteractiveAuthentication yes
```

### `/etc/motley-cue/motley-cue.conf`

Here we configure several different aspects:

- Which OPs do we trust
- Authorisation:
    - Which Virtual Organisations do we support
    - Which individual users do we support
- The privacy statement to display
- Auhorised security staff

The default self-documenting configuration file is shipped with the
`motley-cue` installation.

### `/etc/motley-cue/feudal.conf`

Feudal adapter is the plugin-based tool that implements user provisioning.
Aspects that are configured here include:

- Minimally required levels of assurances
- How to map remote users to local unix accounts
- How to map VO-memberships to local unix groups
- Which backend to use

Again, the configuration file shipped with the `motley-cue` installation
is self explanatory.
