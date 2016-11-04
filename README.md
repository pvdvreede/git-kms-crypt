# git-kms-crypt

Inspired by [git-crypt](https://github.com/AGWA/git-crypt) but tries to simplify the dependencies and use KMS to manage the master key.

git-kms-crypt uses the Git filters and .gitattributes functionality to encrypt files in the git repo on checkin and checkout. It uses AWS KMS data keys to do envelope encryption in conjuction with openssl (this repo does not have any cryptography code in it).

## Dependencies

In trying to keep the requirements light to use this system, the following is required:

* bash
* git
* awscli
* openssl
* an authenticated session with AWS that has access to `GenerateDataKey` and `Decrypt` on a KMS Key

As long as you are not checking in any encrypted files it should gracefully degrade so that people can use repos with encrypted files (but not be able to decypt or re encrypt).

## Usage

The setup requires changes to your `.git/config` file so this repo should be a submodule to a git repo you would like to use it in.

### Initial setup

Run the following to submodule in this repo to the repo you want to use it with (parent repo):

```bash
# make sure you are authenticated with AWS and have the region set where the key is, then:
git submodule add https://github.com/pvdvreede/git-kms-crypt <folder your want to submodule to>
<path to this repo as submodule>/init <aws-key-id-or-alias>
```

This would have created an AWS KMS data-key and put the _encrypted_ version in the `kms.key.enc` file. This is encrypted and safe to commit to your repo. It is used as the master key to encrypt and decrypt your files with openssl.

The setup has also created a new git filter called `kms-crypt` that you can set for different file blogs by creating a `.gitattributes` file in the root of your repo, eg:

```
# .gitattributes
*.mysecretfiles   filter=kms-crypt
```

This would mean that any files ending in `.mysecretfiles` would be encrypted on `git add` and decrypted on `git checkout`.

### Ongoing usage

Once the initial setup has been done, committed and pushed, when other people clone your parent repo out for the first time, they just need to run the following:

```bash
# make sure you are authenticated with AWS and have the region set where the key is, then:
git submodule update --init
<path to this repo as submodule>/init
```

## Example Use case

The idea for this functionality was initialy thought of for using [terraform](https://terraform.io) and encrypting the state files and certain tfvars files that contain secrets.

This should be possible with the following `.gitattributes` files:

```
# .gitattributes
*.tfstate           filter=kms-crypt
*.secrets.tfvars    filter=kms-crypt
```
