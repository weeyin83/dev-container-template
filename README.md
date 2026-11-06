# Azure infrastructure and HTML starter

A reusable, small-scope toolset for Terraform, Bicep, Azure and HTML:

| Environment | How it runs | Internet requirement |
| --- | --- | --- |
| Local Windows + VS Code | Ubuntu 24.04 in WSL 2; no Docker or containers | One-time installation, updates and online services |
| GitHub Codespaces | Ubuntu 24.04 dev container managed by GitHub | Required to create and use a Codespace |

Included: Git, Terraform, standalone Bicep, Azure CLI, GitHub CLI, Live Server,
Terraform/Bicep editor support, Copilot, GitHub pull requests and GitHub Actions
editor support. No optional Node.js runtime, application Python environment,
Docker tooling, Kubernetes, Terragrunt or TFLint. Azure CLI and VS Code extensions
still bring their own required runtime dependencies.

The dev container additionally includes PowerShell, PSScriptAnalyzer and the
PowerShell VS Code extension. These are not installed by the local WSL bootstrap.

This is a development environment, not a deployment pipeline. The Actions extension
helps edit workflows; GitHub CLI can inspect and trigger workflows online. No local
Actions runner or Docker-dependent `act` installation is included.

## Copy into any project

Copy `.devcontainer`, `.vscode`, `scripts`, and `.gitattributes` from this folder
into the root of your repository, and commit them. Merge with existing files rather
than replacing project-specific configuration. Keep this README as setup
documentation, or copy its relevant sections into the project's README.

No project-specific tasks or deployment commands run automatically. This template
does not sign in, deploy resources, or copy credentials.

## Local: one-time online setup

1. In an elevated Windows PowerShell terminal, install WSL if needed:

   ```powershell
   wsl --install -d Ubuntu-24.04
   ```

   Restart Windows if prompted, then launch Ubuntu and create a normal Linux user.
   If you already have this distribution, reuse it; do not reinstall it.

2. Install [VS Code for Windows](https://code.visualstudio.com/) and the
   [WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl).
   Docker Desktop and the Dev Containers extension are not needed for this local path.

3. Keep repositories in the Linux filesystem for better Git and file-watcher
   performance, for example `~/src/my-project`, rather than under `/mnt/c` or OneDrive.
   Copy the template there, then run the installer as your normal user from the
   repository root in an Ubuntu Bash terminal:

   ```bash
   bash scripts/bootstrap-wsl.sh
   ```

   This explicitly online script uses sudo, adds signed HashiCorp, Microsoft and
   GitHub APT repositories, installs their current stable packages, downloads
   Bicep from its official release, and runs local smoke checks. Review it before
   running. It supports Ubuntu 24.04 on x86-64 and ARM64. It does not install WSL
   or change Windows automatically. Rerunning it is an explicit online update,
   not a prerequisite for opening another project.

4. Start a fresh Bash terminal, enter your repository and open VS Code:

   ```bash
   code .
   ```

   Confirm the bottom-left remote indicator says **WSL: Ubuntu-24.04**.
   In Extensions, search `@recommended` and install the workspace recommendations.
   Recommendations are not an automatic installer. Allow VS Code to install each
   extension into the local or WSL extension host it requires.

5. While still online, open an actual `.tf`, `.bicep`, `.html` and workflow `.yml`
   file to activate the relevant extensions and allow any language-server/.NET
   runtime downloads to finish. Confirm Terraform completion and Bicep diagnostics
   work. Sign into GitHub/Copilot if you want their online features.
   VS Code's WSL server must also have been installed for this VS Code version.

After setup, opening a local project does not build a container, run APT, or
download tools. Use **WSL: Reopen Folder in WSL**, not **Reopen in Container**.
For other Bash-based projects in this same distribution, the installed CLIs and
user-wide cache are already available. Other shells must source the equivalent
environment themselves; the setup script configures Bash only.

## HTML Live Server

Open an HTML file and choose **Open with Live Server** or **Go Live**.
Automatic browser launch is disabled to avoid remote-browser problems.

- WSL: open `http://localhost:5500` in your Windows browser. WSL's default localhost
  forwarding makes the Linux server reachable.
- Codespaces: use the **Ports** tab to open port **5500** in your browser. Keep its
  visibility **Private**; only make a port public deliberately.

Live Server binds to loopback, not every network interface. If port 5500 is already
in use, change the workspace setting and the dev container's forwarded port
together. Live Server serves your project files; do not make folders containing
secrets publicly accessible.

## GitHub Codespaces

Commit the template to a GitHub repository, then use **Code > Codespaces >
Create codespace**. To use desktop VS Code, select **Open in Visual Studio Code**
for the Codespace.

The container installs its tools during the build. Its creation hook only creates
user configuration and runs local checks; there are no startup install/update
hooks. VS Code still downloads the requested extensions during first provisioning.
GitHub CLI authentication is generally provided by Codespaces; do not overwrite
its token or commit one. Azure authentication is separate:

```bash
az login --use-device-code
```

PowerShell and PSScriptAnalyzer are installed once at image build time using the
official PowerShell dev container feature. The module is installed for all users;
no module installation or update runs when opening a terminal. Its VS Code
extension is included by the feature. Rebuild an existing container to add them.
The creation hook verifies that the module imports and can analyze a small script
without network access. To rerun that check or analyze your own scripts:

```bash
pwsh -NoLogo -NoProfile -NonInteractive -File scripts/check-powershell.ps1
pwsh -NoProfile -Command 'Invoke-ScriptAnalyzer -Path ./scripts -Recurse'
```

Local WSL users can also sign into GitHub explicitly when online:

```bash
gh auth login
```

For faster **new** Codespaces, enable
[Codespaces prebuilds](https://docs.github.com/en/codespaces/prebuilding-your-codespaces/configuring-prebuilds)
in the repository's settings if available for your account. Prebuilds shift image
build and creation-hook work ahead of opening the environment, but do not promise
instant startup or preinstalled editor extensions. They can consume billable
Actions minutes and storage. Stopping and reopening an existing Codespace avoids
rebuilding its image. A 2-core machine is a reasonable starting point for this
workload; increase only for projects that need it.

This is a lean tool selection, not a tiny image: Azure CLI, Bicep and their editor
support have unavoidable size costs. It intentionally avoids extra language
runtimes, optional infrastructure CLIs and project-wide dependency installs.

## What works offline locally

After all installation and first-use downloads:

- Edit local files, use local Git, serve local HTML, format Terraform, and compile
  Bicep that has no uncached external module dependencies.
- `bash scripts/check-tools.sh` checks CLI versions, Terraform formatting and
  standalone/Azure CLI Bicep availability without signing in or contacting a cloud.
  It is not proof that editor runtimes or a particular project's dependencies are cached.
- Copilot, GitHub API operations, Actions runs, git fetch/push, Azure login,
  deployments and cloud-backed Terraform planning/applying require connectivity.
- A page referencing CDN scripts, fonts or APIs is not fully offline. Keep the
  assets local if offline HTML previews matter.

Prepare **each real project** while online:

```bash
terraform init
az bicep restore --file path/to/main.bicep
```

Use your actual Bicep entry file; restore each entry file with external module
references. Terraform needs its providers, modules and usually remote state
already available. The shared provider cache saves repeated downloads across
projects, but does not make remote backends, uncached modules, or cloud provider
calls work offline. Do not run simultaneous `terraform init` operations against
the same shared plugin cache; Terraform's cache is not concurrency-safe.
Keep `.terraform.lock.hcl` committed; do not commit `.terraform`, Terraform state,
Azure credentials or access tokens.

For offline Terraform validation, first initialize the project's local dependencies
while online (use `terraform init -backend=false` when remote backend setup is not
needed), then use `terraform fmt -check` and `terraform validate` as appropriate.
Offline Bicep compilation with cached external modules can use:

```bash
bicep build path/to/main.bicep --no-restore
```

Before travelling, disconnect temporarily and open the prepared project in WSL.
Exercise Live Server and the actual Terraform/Bicep files you will use. This catches
missing editor runtimes, providers, modules and CDN assets that generic checks cannot.
Do not update VS Code or extensions immediately before offline work unless you
also warm up their new server/runtime downloads.

## Caches, versions and updates

The setup script writes `~/.config/infra-starter/env.sh` and adds a single source line
to `~/.bashrc`. It creates `~/.cache/terraform/plugins`, disables Terraform update
checks and Azure CLI version checks/dynamic extension installation, and directs
Azure CLI to the standalone Bicep binary on PATH. Codespaces receives the same
defaults through `remoteEnv`. These defaults do not block deliberate downloads.
Install any additional Azure CLI extensions explicitly while online.

The initial template deliberately selects current tool releases, not a claimed
fixed version set. Feature major tags and the base-image tag can change; existing
WSL installations and stopped Codespaces retain their installed tools until updated.
If reproducibility matters, pin the base image by digest, feature versions and
individual tool versions, and track corresponding WSL package versions. Codespaces
and WSL share the core infrastructure toolset, not guaranteed byte-identical
installations; PowerShell/PSScriptAnalyzer are dev-container-only additions.

To update WSL tools deliberately while online, rerun the installer. For Codespaces,
rebuild the container to resolve current versions. Neither process upgrades tools
on ordinary project startup. If vendor repositories are already configured on your
WSL installation, inspect/merge duplicate repository entries before running setup.

## References

- [VS Code with WSL](https://code.visualstudio.com/docs/remote/wsl)
- [Azure CLI on Linux](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux)
- [Bicep installation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install)
- [Terraform installation](https://developer.hashicorp.com/terraform/install)
- [GitHub CLI Linux installation](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)
- [Development container Features](https://github.com/devcontainers/features)
