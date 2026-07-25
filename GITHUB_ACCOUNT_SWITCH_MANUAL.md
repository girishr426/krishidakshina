# GitHub Account Switching Manual (Windows + SSH)

This guide shows how to manually switch between multiple GitHub accounts on one machine.

## 1. Check which account is active now

Run:

~~~powershell
ssh -T git@github.com
~~~

If the response says:

- Hi accountA! -> accountA is currently active for github.com
- Hi accountB! -> accountB is currently active for github.com

## 2. Create separate SSH keys (one per account)

If you do not already have separate keys:

~~~powershell
ssh-keygen -t ed25519 -C "your-email-for-accountA" -f "$HOME/.ssh/id_ed25519_accountA"
ssh-keygen -t ed25519 -C "your-email-for-accountB" -f "$HOME/.ssh/id_ed25519_accountB"
~~~

## 3. Add public keys to the correct GitHub accounts

Copy keys:

~~~powershell
Get-Content "$HOME/.ssh/id_ed25519_accountA.pub"
Get-Content "$HOME/.ssh/id_ed25519_accountB.pub"
~~~

Then in each GitHub account:

1. Go to Settings -> SSH and GPG keys
2. Click New SSH key
3. Paste the matching public key

## 4. Configure SSH host aliases

Edit file:

- C:/Users/<your-user>/.ssh/config

Add entries like this:

~~~sshconfig
Host github-accountA
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_accountA
    IdentitiesOnly yes

Host github-accountB
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_accountB
    IdentitiesOnly yes
~~~

## 5. Test each alias

~~~powershell
ssh -T git@github-accountA
ssh -T git@github-accountB
~~~

Each command should greet the expected account.

## 6. Point each repository to the correct alias

Inside the repository folder, set origin to the account that owns the repo.

Example for accountA-owned repo:

~~~powershell
git remote set-url origin git@github-accountA:accountA/repo-name.git
git remote -v
~~~

Example for accountB-owned repo:

~~~powershell
git remote set-url origin git@github-accountB:accountB/repo-name.git
git remote -v
~~~

## 7. Push branch

~~~powershell
git push -u origin your-branch-name
~~~

## 8. Quick manual switch checklist

1. Run ssh -T against the alias you plan to use
2. Verify origin URL uses that alias
3. Push

## 9. Troubleshooting

- Error: Permission denied to wrong user
  - Cause: remote URL points to one account, SSH key authenticates as another
  - Fix: change origin to the correct alias and retest with ssh -T

- Error: Repository not found
  - Cause: account has no access or remote owner/repo is wrong
  - Fix: verify repo path and collaborator permissions

- Wrong key still being used
  - Fix: ensure IdentitiesOnly yes is present in ~/.ssh/config

## 10. Real example from this repository

This repository push worked after switching origin from github.com to a user alias:

~~~powershell
git remote set-url origin git@github-personal:girishr426/krishidakshina.git
git push -u origin feature/mainfiles
~~~

The alias github-personal authenticated as girishr426.