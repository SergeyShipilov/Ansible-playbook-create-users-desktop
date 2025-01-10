Hi, ![](https://user-images.githubusercontent.com/18350557/176309783-0785949b-9127-417c-8b55-ab5a4333674e.gif)my name is Serhii Shypylov
=========================================================================================================================================

-------------------------------

### Socials

<p align="left"> <a href="https://github.com/Shipssv83" target="_blank" rel="noreferrer"> <picture> <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/socials/github-dark.svg" /> <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/socials/github.svg" /> <img src="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/socials/github.svg" width="32" height="32" /> </picture> </a> <a href="https://www.linkedin.com/in/sergey-shipilov-7262a31b4/" target="_blank" rel="noreferrer"> <picture> <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/socials/linkedin-dark.svg" /> <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/socials/linkedin.svg" /> <img src="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/socials/linkedin.svg" width="32" height="32" /> </picture> </a></p>
---

# Ansible Playbook: Create Users Linux Desktop and MacOS

This Ansible playbook automates the creation of users on Linux, macOS. It allows for the configuration of users with various parameters, including username, password, groups, etc.

## Requirements

- Ansible version 2.9 or higher
- Access to managed hosts (Linux, macOS)
- Managed hosts must be configured for remote access (e.g., via SSH for Linux/macOS)

## Variables

You need to define the following variables in your `inventory` or variable file:

- `hosts`: list of servers where the playbook will be applied.
- `username`: name of the first user.
- `comment`: comment for the user (e.g., name or description).
- `password_username`: password for the `username`.
- `password_tmpadmin`: password for the temporary admin `tmpadmin`.

Example:
```yaml
vars:
  username: "newuser"
  comment: "New User Account"
  password_username: "securepassword"
  password_tmpadmin: "adminpassword"
```

Example install-mac.sh:
```bash
#!/bin/sh

fancy_echo() {
  local fmt="$1"; shift
  printf "\n$fmt\n" "$@"
}

sudo -v

fancy_echo "Bootstrapping ..."

trap 'ret=$?; test $ret -ne 0 && printf "failed\n\n" >&2; exit $ret' EXIT

set -e

install_xcode() {
  if ! command -v cc >/dev/null; then
    fancy_echo "Installing Xcode ..."
    xcode-select --install
  else
    fancy_echo "Xcode already installed. Skipping."
  fi
}

install_homebrew() {
  if ! command -v brew >/dev/null; then
    fancy_echo "Installing Homebrew..."
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" </dev/null
  else
    fancy_echo "Homebrew already installed. Skipping."
  fi
}

configure_sudo() {
  if ! sudo grep -q "^$USER ALL=(ALL:ALL) NOPASSWD: ALL$" /etc/sudoers.d/$USER; then
    echo "$USER ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$USER
  else
    fancy_echo "Sudo configuration already exists. Skipping."
  fi
}

setup_ssh() {
  local ssh_dir="/Users/$USER/.ssh"
  local authorized_keys="$ssh_dir/authorized_keys"
  local ssh_keys="ssh-ed25519 ssh-key"

  mkdir -p "$ssh_dir"
  echo "$ssh_keys" > "$authorized_keys"

  chmod 700 "$ssh_dir"
  chmod 600 "$authorized_keys"

  cat "$authorized_keys"
}

install_xcode
install_homebrew
configure_sudo
setup_ssh

fancy_echo "Done."

```

Example install-ubuntu.sh:
```bash
#!/bin/bash

fancy_echo() {
  local fmt="$1"; shift
  printf "\n$fmt\n" "$@"
}

sudo -v

trap 'ret=$?; test $ret -ne 0 && printf "failed\n\n" >&2; exit $ret' EXIT

set -e

configure_sudo() {
  for user in $USER office-adm; do
    if ! sudo grep -q "^$user ALL=(ALL:ALL) NOPASSWD: ALL$" /etc/sudoers.d/$user; then
      fancy_echo "Configuring sudo for user $user..."
      echo "$user ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$user
    else
      fancy_echo "Sudo configuration for $user already exists. Skipping."
    fi
  done
}

install_packages() {
  fancy_echo "Installing necessary packages..."
  sudo apt-get update
  sudo apt-get -y install vim python3 openssh-server ufw
}

setup_ssh() {
  local ssh_dir="/home/$USER/.ssh"
  local authorized_keys="$ssh_dir/authorized_keys"
  local ssh_keys="ssh-ed25519 ssh-key
  fancy_echo "Setting up SSH for user $USER..."

  mkdir -p "$ssh_dir"
  echo "$ssh_keys" > "$authorized_keys"

  chmod 700 "$ssh_dir"
  chown $USER:$USER "$ssh_dir"

  chmod 600 "$authorized_keys"
  chown $USER:$USER "$authorized_keys"

  cat "$authorized_keys"
}

configure_sshd() {
  fancy_echo "Configuring SSH daemon..."

  sudo sed -i 's/^#?PermitEmptyPasswords.*/PermitEmptyPasswords no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?PermitUserEnvironment.*/PermitUserEnvironment no/' /etc/ssh/sshd_config
  sudo sed -i 's/^\(#\s*\)?PasswordAuthentication .*/PasswordAuthentication no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?UsePAM.*/UsePAM no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?X11Forwarding.*/X11Forwarding no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?ClientAliveInterval.*/ClientAliveInterval 300/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?ClientAliveCountMax.*/ClientAliveCountMax 0/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?LogLevel.*/LogLevel VERBOSE/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?MaxAuthTries.*/MaxAuthTries 4/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?IgnoreRhosts.*/IgnoreRhosts yes/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?Protocol.*/Protocol 2/' /etc/ssh/sshd_config
  sudo sed -i 's/^#?Banner.*/Banner \/etc\/issue.net/' /etc/ssh/sshd_config
}

configure_ufw() {
  local ssh_port=22  

  if ! dpkg -l | grep -q ufw; then
    fancy_echo "Installing UFW..."
    sudo apt-get -y install ufw
  fi

  fancy_echo "Configuring UFW for SSH port $ssh_port..."
  sudo ufw allow $ssh_port/tcp comment 'ssh port'
}

restart_ssh_service() {
  if grep -q '^DISTRIB_RELEASE=24.04' /etc/lsb-release; then
    fancy_echo "Ubuntu 24.04 detected. Reloading systemd and restarting ssh..."
    sudo systemctl daemon-reload
    sudo systemctl restart ssh
  else
    fancy_echo "Other Ubuntu version detected. Restarting sshd..."
    sudo systemctl restart sshd
  fi
}

print_network_info() {
  fancy_echo "Network information:"
  ip -o -4 addr show | awk '{print $2, $4}'
}

install_packages
configure_sudo
setup_ssh
configure_sshd
configure_ufw
restart_ssh_service
print_network_info

fancy_echo "Done."

```

# Run the Playbook create user
Run the playbook by specifying your inventory file and variable file:

create user linux
```
ansible-playbook --user root --extra-vars "host=host_name username=user_name password_username=password_user comment='User User'" playbooks/create-users.yml
```

create user macos
```
ansible-playbook --user root --extra-vars "host=host_name new_user_username=username new_user_fullname='User User' new_user_password_cleartext='1111'" playbooks/create-users.yml
```

create user macos tempadmin
```
ansible-playbook --user root --extra-vars "host=host_name new_user_username=tempadmin new_user_fullname='temp admin' new_user_password_cleartext='password' new_user_is_admin=yes" playbooks/create-users.yml
```

Roles
To use this playbook, you need to create a role called create-users. This role should include the tasks to create users based on the provided variables.

License
This project is licensed under the MIT License.