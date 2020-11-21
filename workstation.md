# Workstation

Workstations should be ephemeral. You should be able to re-provision a new workstation within minutes using automation and cloud storage.

### Principles

- Security is always top priority


### Critical Applications

- 1Password - Use a password manager

### Settings

- Passphrase on ssh private keys
- Disk encryption
- `sudo` is rarely/never needed for day-to-day work operations. `npm` example:

  ```
	npm config set prefix ~/.npm
  echo "PATH=$HOME/.npm/bin:$PATH" >> .bashrc



### VS Code

- https://monokai.pro
- ESLint

 
