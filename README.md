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
  connect through bastion/jump hosts (or `ProxyJump` with current versions of
  OpenSSH)

  - For example: [The alternative: ProxyCommand to the rescue - SSH Agent
    Forwarding considered harmful](https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful/#the-alternative-proxycommand-to-the-rescue)

- Use separate SSH Keys for different scopes (including read-only keys).
  Accomplish this by either:

  - Use `solo-agent` to isolate keys when ForwardAgent is needed (ex. for
    remote version control operations)
  - Or install the SSH key pair (ex. for remote version control operations) on
    the remote host. This option is less secure, but also less complex.


## Using `solo-agent`

1. Assumptions:

   - You need to access GitHub from a host (`devhost`) on which a third-party
     has root access
   - You have already created a SSH key pair for use with GitHub and added to
     your GitHub account as a read-only key
   - The private key mentioned above is located on your laptop at:
     `~/.ssh/rsa_github_ro`
   - You have cloned this repository to to your laptop. It is located at:
     `~/git/solo-agent`

2. At the top of your SSH configuration, put the Match exec that starts the
   SSH agent:

        # vim: set ft=sshconfig
        Match exec "~/git/solo-agent/solo-agent github_ro rsa_github_ro"

3. In the middle of your SSH configuration, put the `devhost` stanza:

        Host devhost
            HostName devhost.example.com
            ForwardAgent Yes
            IdentityAgent ~/.ssh/solo-sock/github_ro

4. At the bottom of your SSH configuration, ensure the global `Host *` stanza
   includes the following two options:

        Host *
            AddKeysToAgent no
            ForwardAgent no


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


## Requirements

- [OpenSSH 7.3](https://www.openssh.com/txt/release-7.3) added `IdentityAgent`:

  - macOS 10.13 High Sierra or later
  - Red Hat Enterprise Linux 7 Update 4 or later
  - Ubuntu 17.04 Zesty Zapus or later

- Either:

    - GNU coreutils readlink
    - Python


## Alternatives

- Install discrete and properly scoped SSH key pairs on the remote host
- [Managing multiple SSH agents - Wikitech](https://wikitech.wikimedia.org/wiki/Managing_multiple_SSH_agents)


## License

- [LICENSE](LICENSE) (Expat/[MIT License][MIT])

[MIT]: http://www.opensource.org/licenses/MIT "The MIT License (MIT)"
