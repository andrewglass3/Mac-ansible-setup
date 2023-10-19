<img src="https://raw.githubusercontent.com/geerlingguy/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png" width="250" height="156" alt="Mac Dev Playbook Logo" />

# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

This repo/playbook is forked from Jeff Geerling https://github.com/geerlingguy/mac-dev-playbook

## Installation

1. Ensure Apple's command line tools are installed (`xcode-select --install` to launch the installer).
2. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html):

   1. Run the following command to add Python 3 to your $PATH: `export PATH="$HOME/Library/Python/3.9/bin:/opt/homebrew/bin:$PATH"`
   2. Upgrade Pip: `sudo pip3 install --upgrade pip`
   3. Install Ansible: `pip3 install ansible`

3. Clone or download this repository to your local drive.
4. Change your username in the following files to match your mac username:
    main.yml - lines 38, 40
    tasks/osx.yml - lines 7 and 11. ** NOTE ** If you are running an intel mac - comment out lines 8 to 11 as rosetta is only for Apple Silicone Macs
4. Run `ansible-galaxy install -r requirements.yml` inside this directory to install required Ansible roles.
5. Run `ansible-playbook main.yml --ask-become-pass` inside this directory. Enter your macOS account password when prompted for the 'BECOME' password.

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

1. (On the Mac you want to connect to:) Go to System Preferences > Sharing.
2. Enable 'Remote Login'.

> You can also enable remote login on the command line:
>
>     sudo systemsetup -setremotelogin on

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

```
[ip address or hostname of mac]  ansible_user=[mac ssh username]
```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

## Overriding Defaults

Not everyone's development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

```yaml
homebrew_installed_packages:
  - cowsay
  - git
  - go

mas_installed_apps:
  - { id: 967805235, name: "Paste" }
  - { id: 1451685025, name: "Wireguard" }
  - { id: 409203825, name: "Numbers" }
  - { id: 1153157709, name: "Speedtest by Ookla" }

composer_packages:
  - name: hirak/prestissimo
  - name: drush/drush
    version: "^8.1"

gem_packages:
  - name: bundler
    state: latest

npm_packages:
  - name: webpack

pip_packages:
  - name: mkdocs

configure_dock: true
dockitems_remove:
  - Launchpad
  - TV
dockitems_persist:
  - name: "Sublime Text"
    path: "/Applications/Sublime Text.app/"
    pos: 5
```

Any variable can be overridden in `config.yml`; see the supporting roles' documentation for a complete list of available variables.

## Included Applications / Configuration (Default)

Applications (installed with Homebrew Cask):

- [ChromeDriver](https://sites.google.com/chromium.org/driver/)
- [Firefox](https://www.mozilla.org/en-US/firefox/new/)
- [Google Chrome](https://www.google.com/chrome/)
- [Homebrew](http://brew.sh/)
- [Slack](https://slack.com/)
- [Sublime Text](https://www.sublimetext.com/)
- [Transmit](https://panic.com/transmit/) (S/FTP client)
- [iterm2](https://iterm2.com) (Replacement Terminal)
- [1Password](https://1password.com) (Password Manager)
- [Visual Studio Code](https://code.visualstudio.com) (Code Editor IDE)
- [Viscosity](https://www.sparklabs.com/viscosity/)(Openvpn clinet software)
- [Notion](https://www.notion.so)(Evernote like app)
- [Fig](https://fig.io) (Intelligent terminal prompt assistant)
- [Rectangle](https://rectangleapp.com) (Window snapping and management app)
- [font-hack-nerd-font](https://www.nerdfonts.com) (Font for terminal)
- [font-fira-code-nerd-font](https://www.nerdfonts.com) (Font for terminal)
- [font-meslo-for-powerline](https://www.nerdfonts.com) (Font for terminal)

Packages (installed with Homebrew):

- autoconf
- awscli
- bash-completion
- git
- github/gh/gh
- go
- gpg
- iperf
- kubectx
- MonitorControl
- nmap
- ssh-copy-id
- readline
- openssl
- wget
- zsh-history-substring-search
- zsh
- zsh-autosuggestions
- zsh-completions

- chromedriver
- docker
- firefox
- google-chrome
- iterm2
- sequel-ace
- slack
- sublime-text
- transmit
- 1password
- visual-studio-code
- font-hack-nerd-font
- font-fira-code-nerd-font
- viscosity
- notion
- font-meslo-for-powerline
- fig
- rectangle

My [dotfiles](https://github.com/andrewglass3/dotfiles) are also installed into the current user's home directory, including the `.osx` dotfile for configuring many aspects of macOS for better performance and ease of use. You can disable dotfiles management by setting `configure_dotfiles: no` in your configuration.

Finally, there are a few other preferences and settings added on for various apps and services.

## Full / From-scratch setup guide

Since I've used this playbook to set up something like 20 different Macs, I decided to write up a full 100% from-scratch install for my own reference (everyone's particular install will be slightly different).

You can see my full from-scratch setup document here: [full-mac-setup.md](full-mac-setup.md).

## Testing the Playbook

Many people have asked me if I often wipe my entire workstation and start from scratch just to test changes to the playbook. Nope! This project is [continuously tested on GitHub Actions' macOS infrastructure](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI).

You can also run macOS itself inside a VM, for at least some of the required testing (App Store apps and some proprietary software might not install properly). I currently recommend:

- [UTM](https://mac.getutm.app)
- [Tart](https://github.com/cirruslabs/tart)

## Ansible for DevOps

Check out [Ansible for DevOps](https://www.ansiblefordevops.com/), which teaches you how to automate almost anything with Ansible.

## Author

This project was created by [Jeff Geerling](https://www.jeffgeerling.com/) (originally inspired by [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks)).

[badge-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/workflows/CI/badge.svg?event=push
[link-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI
