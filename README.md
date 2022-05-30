# GEE - Git Encryption Extension

- Author: Christian Engel © 2022 mailto:engel-ch@outlook.com
- Last update date: --u2022-05-30
- Creation date: --c2021-05-07
- Classification: public
- Described GEE version: 1.0.0

## Contents

- [GEE - Git Encryption Extension](#gee---git-encryption-extension)
  - [Contents](#contents)
  - [Summary](#summary)
  - [Introduction, Terminology](#introduction-terminology)
  - [GEE Requirements](#gee-requirements)
  - [Analysis](#analysis)
  - [Installation](#installation)
    - [Technical Dependencies](#technical-dependencies)
    - [Putting a git Project under GEE](#putting-a-git-project-under-gee)
    - [Integrating GEE into git](#integrating-gee-into-git)
    - [ConfigShell Integration](#configshell-integration)
    - [Checking Installation](#checking-installation)
  - [Usage](#usage)
    - [gee.cfg](#geecfg)
    - [Temporarily disabling gee.cfg for a Commit](#temporarily-disabling-geecfg-for-a-commit)
    - [Encrypting Files](#encrypting-files)
    - [Encryption Details](#encryption-details)
    - [Encryption Vault Ids](#encryption-vault-ids)
    - [Decrypting Files](#decrypting-files)
    - [Show encryption status of private files](#show-encryption-status-of-private-files)
    - [Show encryption status details](#show-encryption-status-details)
  - [Example Case](#example-case)

## Summary

- Symmetric AES256-based (using ansible-vault) encryption utility
- Making sure that files cannot be committed if they are not in encrypted form
- based on bash
- Easiest installation via [ConfigShell](https://github.com/Engelch/ConfigShell) (git-submodule)
- MIT license

## Introduction, Terminology

GEE, the git encryption extension, shall ensure that you can never accidentally commit secret files in an unencrypted state. The current version 1 uses symmetric encryption to en- or decrypt files. The passphrase is to be shared by a multi-user password management tool outside of GEE's scope. The terms encryption key, password, or passphrase are here understood to be interchangeable. Minimal password strength requirements should be defined such as:

1. length ≥ 32 characters
2. passphrase has consist of a mixture of lower and upper case characters, digits, and special characters

In many projects, we have to take care of credentials we name from here on either private or secret files. The term secret files are also understood to apply to a directory. In such a case, all its included files (also in potential subdirectories) are considered to be secret files. GEE concentrates on file-contents encryption; it encrypts neither directory- nor file names.

GEE must be easy to install and use.

## GEE Requirements

1. Guiding the user: even if you forget about the commands, the system prevents you making big mistakes (e.g. publishing secret files in an unencrypted manner)
2. Short learning curve: usable after reading this document or after a 15 minutes introduction.
3. Implement KISS Principle. The code must be easy to understand and to review.
4. Easy and fast to install
5. The software is supposed to be OSS. So larger the user-group so easier to maintain and improve the solution.
6. GEE must at least run under Linux, WSL, OSX.

## Analysis

We experimented with tools such as `git-secret`, `git-secrets`, and `git-crypt`. However, either these tools were too complex for us, or they allowed us to commit the private files in unencrypted form. For this reason, I looked for another solution and finally developed gee.

## Installation

Please make sure that the technical dependencies are installed. Furthermore, they should be reachable by your PATH environment variable.

### Technical Dependencies

GEE V.1 requires:

- bash
- git
- ansible-vault
- find
- sed
- egrep
- wc
- dirname, basename

I.e. on a regular UNIX-based system, make sure that you installed git, ansible-vault, and bash. Furthermore, I recommend [git-flow](https://danielkummer.github.io/git-flow-cheatsheet/)) for feature-based development.

### Putting a git Project under GEE

I recommend to install a `gee` directory or a  `git-hooks` directory in your repository which holds the file `pre-commit`. Then, I create an s-link as:

  ```bash
  cd .git/hooks
  ln -s ../../git-hooks/pre-commit
  ```

It is also possible to load gee as a *git submodule* into your repository but as
this is not known to every git user, a more simplified solution as described above can make sense. Furthmore, it then also directly denotes which version of *git gee* was used by your project.

After that and if gee was installed on you system, the project is ready to use gee.

### Integrating GEE into git

To be able to do `git gee ...` calls, the directory containing `gee` (and the sub-commands `gee-encryptPrivateFiles`, `gee-privateFiles`,
`gee-decryptPrivateFiles`, `gee-privateEncryptedFiles`, and `gee-privateUnencryptedFiles`) must be mentioned by the PATH environment variable.
This is automatically done when GEE is installed via the [ConfigShell](https://github.com/Engelch/ConfigShell) repository.

### ConfigShell Integration

This is the recommended installation way.

GEE is integrated as a git-submodule into [ConfigShell](https://github.com/Engelch/ConfigShell). ConfigShell is a github repository with default bash, zsh, ... files that we install on many servers around the world. When using ConfigShell, you can also link the `pre-commit` command to `.../ConfigShell/gee/gee/pre-commit`. ConfigShell is of course OSS and published under the MIT license.

### Checking Installation

1. make sure, the `pre-commit` file exists in your `.git/hooks/` directory and it points to a file in case of an s-link.

    As .git is not directly part of the git repository, hooks are also not stored in a remote push. Therefore, it is recommended to create a `git-hooks` directory in your repository. This hints other users that something has to be linked to work as expected. The usual setup command would be:

2. start the pre-commit manually by entering something like `.git/hooks/pre-commit`

    The following output is expected: `ERROR:pre-commit hook installed but no gee.cfg files`
3. Now, enter the command `git gee list` and you should see the same error message as under 2.

Congratulations, gee is installed and working. Now, we have to define which files are private ones.

## Usage

### gee.cfg

The `gee.cfg` files are text files. These configuration files can exist anywhere in the git project tree. There contents is supposed to contain relative file-patterns and shall be used with the semantics in mind that the are valid from this directory on. It can be easier just to maintain one configuration file in the git root repository directory, but this is up to you. Here an example:

```bash
test/a.txt
test/test4/
test/test5/*.cfg
```

The lines in such a configuration file are used as name arguments to the find(1) command. Empty lines and lines starting with a `#` hash-mark are omitted. If a specified files does not exist,
then `pre-commit` returns an error and prevents committing.

### Temporarily disabling gee.cfg for a Commit

If you are absolutely sure what you are doing and you want to commit something - hopefully not private files - then you can call commit like

```bash
git commit --no-verify
```

With this option, hooks are not executed.

### Encrypting Files

The `git gee` command (actually `gee`) is an extension for the *gee pre-commit hook*.
It is possible to use gee with just the hook.

You first need a pasword. The password is considered to be in a file which can be wiped, e.g. using the `wipe` or `shred` command. You can encrypt all files using:

```bash
git gee encrypt ./testpw1.txt

git gee e ./testpw1.txt # equivalent
```

Instead of encrypt, you could also just say: e, en, enc, encr, encry, encryp

### Encryption Details

`gee` actually calls `gee-encrypt` for either en- or decryption. This command also offers a help mode which can be triggered by supplying the `-h` option.

`gee-encrypt` also supports the encryption of files specified on the command line (forced mode). This can be used to encrypt certain files with different passwords.

### Encryption Vault Ids

If you specify a file like ../testpw1.txt as the passphrase file, then *testpw1* will be used in the encrypted files as the vault id. This can be used as a hint to specify which encryption password was used. If you want to specify you own vault id, you can specify this in the form

```bash
vault-id@<<path-to-passphrase-filename>>
```

### Decrypting Files

Use

```bash
git gee decrypt ./testpw1.txt

git gee d testpw1.txt  # shorter and equivalent
```

Instead of decrypt, you could also just say: d, de, dec, decr, decry, decryp, u, un, une, ..., unencrypt.

### Show encryption status of private files

You can use

```bash
git gee list
```

Instead of list, you can also say l, li, lis, lst

### Show encryption status details

The `gee` command uses the `gee-privateFiles` to list all files under gee. This command also offers a help mode which can be triggered by supplying the `-h` option.

## Example Case

Let's check the status of the example directory `test/` in the git gee repository.

```bash
/opt/p/gee$ ./bin/gee l
/opt/p/gee/test/a.txt encrypted testpw1
/opt/p/gee/test/test2//b.txt encrypted testpw1
/opt/p/gee/test/test3/3.cfg encrypted testpw1
/opt/p/gee/test/test3/4.cfg encrypted testpw1
/opt/p/gee/test/test3/5.cfg encrypted testpw1
/opt/p/gee/test/test3/6.cfg encrypted testpw1
```

We can see that:

1. All files are encrypted
2. All files are encrypted with the same vault-id, which is testpw1. As a file with the name testpw1.txt exists (which should usually **NEVER** be part of the repository), we can assume that this file contains the passphrase.

We can look at the `testpw1.txt` file, but it does not really help:

```bash
cat testpw1.txt 
a52e68095d40ace5e4dbad7e2e8c5fb3f9cc37a8731470e8520e6e3ac7a23967
```

I created the file using `echo $RANDOM | gosha256 > testpw1.txt`.

Now, let's unencrypt the files with:

```bash
./bin/gee d testpw1.txt
Decryption successful
Decryption successful
Decryption successful
Decryption successful
Decryption successful
Decryption successful
```

Decryption was working. The files are replaced in place with their unencrypted counterparts. Now, let's try to commit the unencrypted versions.

```bash
git add test/
git commit -m 'commit new versions in unencrypted form. This should not be possible'
Unencrypted private file found: /Users/engelch/X/gee/test/a.txt
Unencrypted private file found: /Users/engelch/X/gee/test/test2//b.txt
Unencrypted private file found: /Users/engelch/X/gee/test/test3/3.cfg
Unencrypted private file found: /Users/engelch/X/gee/test/test3/4.cfg
Unencrypted private file found: /Users/engelch/X/gee/test/test3/5.cfg
Unencrypted private file found: /Users/engelch/X/gee/test/test3/6.cfg
```

The exit code is unequal to 0. We could not commit them. Now, let's encrypt them:

```bash
./bin/gee e testpw1.txt 
Encryption successful
Encryption successful
Encryption successful
Encryption successful
Encryption successful
Encryption successful
git add test/
git commit -m 'encrypted version, no contents change'
[master 97cd026] encrypted version, no contents change
 7 files changed, 38 insertions(+)
 create mode 100644 test/a.txt
 create mode 100644 test/gee.cfg
 create mode 100644 test/test2/b.txt
 create mode 100644 test/test3/3.cfg
 create mode 100644 test/test3/4.cfg
 create mode 100644 test/test3/5.cfg
 create mode 100644 test/test3/6.cfg
 ```

 Now, we can commit the files. Finally, let's clean up the wrong commit:

 ```bash
 git reset HEAD~
 ```

 This should you help starting. As a last topic, let's add `gee` to `git`.

 ```bash
PATH=$PATH:$(git rev-parse --show-toplevel)/bin
git gee l
/Users/engelch/X/gee/test/a.txt encrypted testpw1
/Users/engelch/X/gee/test/test2//b.txt encrypted testpw1
/Users/engelch/X/gee/test/test3/3.cfg encrypted testpw1
/Users/engelch/X/gee/test/test3/4.cfg encrypted testpw1
/Users/engelch/X/gee/test/test3/5.cfg encrypted testpw1
/Users/engelch/X/gee/test/test3/6.cfg encrypted testpw1
```

As `gee` is now in the PATH we can call either `gee` directly or as `git gee`. Happy hacking!




