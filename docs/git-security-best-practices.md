# Git Security Best Practices

## SSH auth

### Managing SSH keys

1. Generate a different SSH key for each hosted server (e.g. github.com vs GitHub enterprise)

    ```shell
    ssh-keygen -i ~/.ssh/id_${HOST}
    ```

2. Use a significantly complex passphrase for the private key. It is recommeded that you generate the passphrase from a password manager and store the generated key and the passphrase in the password manager.
3. Configure SSH to use the generated key when accessing the host by providing a configuration in `~/.ssh/config`. E.g. to access github with a particular key use the following:

    ```shell
    Host github.com
      HostName github.com
      User git
      IdentityFile /path/to/id/file
      IdentitiesOnly yes
      AddKeysToAgent yes
      UseKeychain yes
    ```

4. Add the public key to your user setting in your git server. For GitHub the instructions can be found here - https://docs.github.com/en/enterprise-cloud@latest/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
5. Add your SSH key to the ssh-agent

    ```shell
    eval "$(ssh-agent -s)"
    ssh-add --apple-use-keychain /path/to/id/file
    ```

**Note:** It is a good idea to rotate the keys and/or the passphrase occationally. There is nothing (yet) that will enforce this in GitHub so you will need to add a reminder to rotate the SSH keys.

### (Optionally) Managing SSH keys with 1Password

1. Create a Login entry
2. Store and/or generate the passphrase in the password
3. Add the private key and public key files as files in the entry. **Note**: to enable 1Password to read the keys from the hidden `.ssh` directory, you may temporarily need to create a public sym link - `ln -s ~/.ssh ~/ssh`

## Signing commits

### Prerequisites

Install the gpg keys:

```shell
brew install gpg gpg2 pinentry-mac
```

### Generate a new GPG key

1. Generate GPG keys. Take the defaults for all prompted values and provide a sufficiently complex passphrase.

    ```shell
    gpg --full-generate-key
    ```

2. List the keys

    ```shell
    gpg --list-secret-keys --keyid-format=long
    ```

3. From the list of the GPG keys, copy the long form of the GPG key ID you'd like to use. In this example, the GPG key ID is `3AA5C34371567BD2`:

    ```shell
    /Users/hubot/.gnupg/secring.gpg
    ------------------------------------
    sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
    uid                          Hubot 
    ssb   4096R/42B317FD4BA89E7A 2016-03-10
    ```

4. Export the public GPG key by providing the long key ID from the previous step:

    ```shell
    gpg --armor --export ${GPG_KEY_ID}
    ```

5. Copy your GPG key, beginning with `-----BEGIN PGP PUBLIC KEY BLOCK-----` and ending with `-----END PGP PUBLIC KEY BLOCK-----`

6. [Add the GPG key to your GitHub account](https://docs.github.com/en/github-ae@latest/articles/adding-a-new-gpg-key-to-your-github-account)

### Add the GPG passphrase to the keychain

1. Use the `pinentry-mac` command to add the passphrase to the keychain

    ```shell
    mkdir -m 0700 ~/.gnupg
    echo "pinentry-program $(brew --prefix)/bin/pinentry-mac" | tee ~/.gnupg/gpg-agent.conf
    pkill -TERM gpg-agent
    ```

2. Close and reopen the shell

3. List the keys

    ```shell
    gpg --list-keys
    ```

4. Test the passphrase stored in keychain. Use the email address identity from the `uid` line in the previous command.

    ```shell
    echo test | gpg -e -r ${EMAIL_IDENTITY} | gpg -d
    ```

### Set up git to use the GPG key by default

1. List the secret keys with `gpg --list-secret-keys --keyid-format=long`

    ```shell
    /Users/hubot/.gnupg/secring.gpg
    ------------------------------------
    sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
    uid                          Hubot 
    ssb   4096R/42B317FD4BA89E7A 2016-03-10
    ```

2. Add the global signing key to the git config. (In the previous example the signing key is `3AA5C34371567BD2`)

    ```shell
    git config --global user.signingkey 3AA5C34371567BD2
    git config --global tag.forceSignAnnotated true
    git config --global gpg.program /usr/local/bin/gpg
    git config --global commit.gpgsign true
    ```


### (Optionally) Managing GPG keys in 1Password

1. Create a login entry in 1Password

2. Add the passphrase in the password field

3. Export the public and private GPG keys

    ```shell
    gpg --output public.gpg --armor --export ${GPG_KEY_ID}
    gpg --output private.gpg --armor --export ${GPG_KEY_ID}
    ```

4. Add the public and private GPG keys to the login entry


## Resources

- [GPG key instructions](https://docs.github.com/en/github-ae@latest/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)

