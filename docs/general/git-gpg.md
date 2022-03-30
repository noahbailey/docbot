# Git GPG signing

## Create a key

If you don't have a key already, create one: 

    gpg --full-generate-key

The key should be RSA/RSA, 4096 bits, and expire in 2-5 years. 

View the key: 

    gpg -k

```
gpg --list-secret-keys


/home/ongo/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096 2020-09-15 [SC] [expires: 2025-09-14]
      988881adc9fc3655077dc2d4d757d480b5ea0e11

uid           [ unknown] Ongo Gablogian (https://gablogian.org) <ongo@gablogian.org>
sub   rsa4096 2020-09-15 [E] [expires: 2025-09-14]
```

## Export public key

The public key will need to be submitted to your identity provider, for example added to your known keys in GitHub. 

    gpg --armor --export 988881adc9fc3655077dc2d4d757d480b5ea0e11

## Configure Git

Configure your git client to always sign commits: 

    git config --global user.name "Ongo Gablogian"
    git config --global user.email "ongo@gablogian.org"
    git config --global user.signingkey "988881adc9fc3655077dc2d4d757d480b5ea0e11"
    git config --global commit.gpgsign true
