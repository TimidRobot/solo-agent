[![badge: Supported by Clockwork](https://img.shields.io/badge/Supported%20by-Clockwork-ffcc00.svg)](https://www.clockwork.com/)

# solo-agent

Enable discrete SSH Agents to avoid leaking access across hosts


## SSH Agent Security Issues


### ForwardAgent Issue

- Anyone with root access on a remote host you have forwarded your SSH Agent
  to can use the agent socket to access any host you have authorized that key
  on and can eavesdrop on your ongoing session.


### ForwardAgent Resolution

- Do not forward your SSH Agent. You do not need it to use `ProxyCommand` to
  connect through bastion/jump hosts.

  - For example: [The alternative: ProxyCommand to the rescue - SSH Agent
    Forwarding considered harmful](https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful/#the-alternative-proxycommand-to-the-rescue)

- Use seperate SSH Keys for different scopes (including read-only keys).
  Accomplish this by either:

  - Use `solo-agent` to isolate keys when ForwardAgent is needed (ex. for
    remote version control operations)
  - Or install the SSH key pair (ex. for remote version control operations) on
    the remote host. This option is less secure, but also less complex.


## Using `solo-agent`

1. Assumtions

   - You need to access GitHub from a host (`devhost`) on which a third-party
     has root access
   - You have configured a GitHub read-only key and it's located on your
     laptop at: `~/.ssh/rsa_github_ro`
   - You have cloned this repository to `~/git/solo-agent`

2. Ensure the global section of your SSH configuration contains:

        Host *
            AddKeysToAgent no
            ForwardAgent no

3. Add ssh include: `~/.ssh/inc_github_ro`:

        # vim: set ft=sshconfig
        ForwardAgent Yes
        IdentityAgent ~/.ssh/solo-sock/github_ro
        Match exec "~/git/solo-agent/solo-agent github_ro rsa_github_ro"

4. Update the `devhost` Host stanza to use the include:

        Host devhost
            HostName devhost.example.com
            Include inc_github_ro


### Explanation

When you `ssh devhost` with the configuration above, the following will happen:
1. The Match directive in the include will execute `solo-agent`. It will
   determine if there is already a valid socket symlinked from
   `~/.ssh/solo-sock/github_ro`:

   - If there is, it will ensure the specified key is loaded into that agent
   - If not, it will start a new agent, create the symlink, and ensure the
     specified key is loaded into that agent

2. The SSH connection to `devhost` will use the SSH Agent connected to the
   specified socket. Only the key(s) added to it will be available.

   - You can continue to authenticate to `devhost` with the `IdentityFile` of
     your choice without worry.


## License

- [LICENSE](LICENSE) ([MIT License][MIT])

[MIT]: http://www.opensource.org/licenses/MIT "The MIT License (MIT)"
