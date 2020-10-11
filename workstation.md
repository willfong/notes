# Workstation

Workstations should be ephemeral. You should be able to re-provision a new workstation within minutes using automation and cloud storage.


### Critical Applications

- 1Password - Use a password manager

### Settings

- Passphrase on ssh private keys
- Disk encryption
- `sudo` is rarely/never needed for day-to-day work operations. `npm` example:

  ```npm config set prefix ~/.npm
  echo "PATH=$HOME/.npm/bin:$PATH" >> .bashrc

