# GPG

## Generate a key

```
gpg --full-generate-key
gpg (GnuPG) 2.2.28; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
Key expires at Sun 16 Jul 2023 09:47:46 PM EDT
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Ongo Gablogian
Email address: ongo@gablogian.org
Comment:
You selected this USER-ID:
    "Ongo Gablogian <ongo@gablogian.org>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
```

## Upload key

Upload to keyserver

    gpg --send-key DEADBEEF


