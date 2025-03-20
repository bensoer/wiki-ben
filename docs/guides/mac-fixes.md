# Mac Fixes

## Disable Git Credential Caching From OSX keychain

### Problem
Receive 403s from git when applying actions to CodeCommit or GHE accounts. Seemingly happens spontaniously, though generally occurs every 8-24hrs

### Cause
This problem occurs when switching between numerous CodeCommit and GHE accounts. The error that happens is also extremely unclear. You simply just get 403's all the time. Its because git by default caches the credentials being used, but credentials being used for GHE and CodeCommit are temporary and expire anywhere from 8 to 24hrs. 

### Quick Fix
The quick fix solution is to delete the entries from keychain:

1. Type CMD + Spacebar to open search
2. Search "keychain" and hit enter
3. Select "login" under "Keychains" on the top right menu
4. Select "All Items" under "Category" on the bottom right menu
5. In the search on the top right search "codecommit", "git", and "ghe"
6. Delete all the entries returned from those search queries
7. All cached credentials for git have now been removed

### Permanent Fix
There are a couple of ways to apply this. A previous trick by Martin Ho in his docs has solved this originally : Stuff Martin Knew#GitConfiguration . But this solution only works if you are then manually configuring git's /.git/config for every single repository you are cloning. When wanting to take advantage of git's more default configurations and hierarchy, more needs to be done. Complete the following to configure git to stop using OSX Keychain cache:

1) Open terminal and run the following command
```bash
git config --get-all --show-origin credential.helper
```
This will show you all the layers of configuration of the credential.helper module. You will likely see the following entry:
```
file:/usr/local/git/etc/gitconfig       osxkeychain
```
2) Open the file `/usr/local/git/etc/gitconfig` and comment out the following line
```toml
[credential]
	helper = osxkeychain
```
3) Save changes and close

4) Open the file `/Library/Developer/CommandLineTools/user/share/git-core/gitconfig` and comment out the following line
```toml
[credential]
	helper = osxkeychain
```
5) Save changes and clone

6) (Optional) Add as an added bonus. I would recommend still applying martin's fix to enforce a default "no credential helper defined" across all projects. This can be done simply by putting the following code block at the top of your `/etc/gitconfig` and `~/.gitconfig` files:
```toml
[credential]
	helper = ""
```
Any credential blocks created further down the `/etc/gitconfig` or `~/.gitconfig` files will still be valid, but by default nothing created before hand will be valid. This will help enforce removal of any default osxkeychain caching

7) Rerun the following command to verify changes were made. You should no longer see the oskeychain entry in the response
```bash
git config --get-all --show-origin credential.helper
```
