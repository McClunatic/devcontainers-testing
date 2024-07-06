# Dev Containers testing

Simple repository for testing with the
[devcontainers CLI](https://github.com/devcontainers/cli).

## Recreating this repository

1. Install the devcontainers CLI, for example
   [using npm install](https://github.com/devcontainers/cli?tab=readme-ov-file#npm-install).
2. Use a devcontainers template to create the
   [.github/dependabot.yml](.github/dependabot.yml) and
   [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json) contents.
   See below for an example to get started:

```sh
❯ devcontainer templates apply \
    --template-id ghcr.io/devcontainers/templates/python:3.1.0 \
    --template-args '{"imageVariant": "3.12-bookworm"}' \
    --features '[
      {"id": "ghcr.io/devcontainers/features/sshd:1", "options": {"username": "vscode", "sshd_port": 10648, "start_sshd": true}},
      {"id": "ghcr.io/devcontainers-contrib/features/starship:1"}
    ]'
```
3. Modify the 
   [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json) to add:
     * `workspaceFolder`: Path to the workspace folder in the container.
     * `workspaceMount`: Mount parameter for `docker run`. Target should match
       `workspaceFolder`.
     * `postCreateCommand`: Command to run after the container is created.
       In this example it is set to `sudo chown vscode:vscode /workspace` to
       ensure the mounted volume is owned by the `remoteUser`, `vscode`.
     * `appPort`: Specifies the port(s) to expose from the container. Should
       be set to `10648` if using the example options above for `sshd`.
4. Start the devcontainer:

```sh
❯ devcontainer up --workspace-folder .`.
...
{"outcome":"success","containerId":"d3982d159758e87ea49d20947e14e7ee5a7e3eb8dfd92f712e285785f7a516f3","remoteUser":"vscode","remoteWorkspaceFolder":"/workspace"}
```
> If you have a dotfiles repository, you can use them within the devcontainer
> by appending `--dotfiles-repository [URL]`. For example,
> `--dotfiles-repository https://github.com/McClunatic/dotfiles.git`.
> To see more information about using dotfiles with devcontainers, use
> `devcontainer up --help`.

5. Authorize a local ssh key for the `vscode` user to enable authentication:

```sh
❯ ssh-add -L | devcontainer exec --workspace-folder . bash -c "umask 77 && mkdir -p ~/.ssh && cat - >> ~/.ssh/authorized_keys"
```
> The above example presumes an `ssh-agent` is running and that keys can
  be queried using `ssh-add -L`. An alternative is to `cat` a public key
  directly from file.

6. Connect to the container:

```sh
❯ ssh -p 10648 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o GlobalKnownHostsFile=/dev/null -A vscode@localhost
Warning: Permanently added '[localhost]:10648' (ED25519) to the list of known hosts.
Linux d3982d159758 5.15.153.1-microsoft-standard-WSL2 #1 SMP Fri Mar 29 23:16:34 UTC 2024 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
vscode in 🌐 fd80231983d4 in ~
⬢ [Docker] ❯
```
As noted in the
[sshd feature README](https://github.com/devcontainers/features/blob/main/src/sshd/NOTES.md),
the `-o`  arguments are optional, but will prevent you from getting warnings or
errors about known hosts when you do this from multiple containers/codespaces.
