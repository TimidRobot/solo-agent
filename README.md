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
    Forwarding considered harmful][harmful]
- Use separate SSH Keys for different scopes (including read-only keys).
  Accomplish this by either:
  - Use `solo-agent` to isolate keys when ForwardAgent is needed (ex. for
    remote version control operations)
  - Or install the SSH key pair (ex. for remote version control operations) on
    the remote host. This option is less secure, but also less complex.

[harmful]:https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful/#the-alternative-proxycommand-to-the-rescue


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
   - You have symlinked `solo-agent` to `~/bin/solo-agent`
2. At the top of your SSH configuration, put the Match exec that starts the
   SSH agent:
    ```
    Match exec "~/bin/solo-agent github_ro rsa_github_ro"
    ```
3. In the middle of your SSH configuration, put the `devhost` stanza:
    ```
    Host devhost
        HostName devhost.example.com
        ForwardAgent Yes
        IdentityAgent ~/.ssh/solo-sock/github_ro
    ```
4. At the bottom of your SSH configuration, ensure the global `Host *` stanza
   includes the following two options:
    ```
    Host *
        AddKeysToAgent no
        ForwardAgent no
    ```


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

- [OpenSSH 7.3][openssh73] added `IdentityAgent`:
  - macOS 10.13 High Sierra or later
  - Red Hat Enterprise Linux 7 Update 4 or later
  - Ubuntu 17.04 Zesty Zapus or later
- Either:
  - GNU coreutils readlink
  - Python

[openssh73]: https://www.openssh.com/txt/release-7.3


## Alternatives

- Install discrete and properly scoped SSH key pairs on the remote host
- [Managing multiple SSH agents - Wikitech][multissh]

[multissh]: https://wikitech.wikimedia.org/wiki/Managing_multiple_SSH_agents


## Supported By

Portions of the development of this project were supported by
[ClockworkNet][Clockwork]. Thank you!

[Clockwork]: https://github.com/ClockworkNet


## License

- [`LICENSE`](LICENSE) (Expat/[MIT][mit] License)

[mit]: http://www.opensource.org/licenses/MIT "The MIT License | Open Source Initiative"
