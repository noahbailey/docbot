# SSH Keys

## Generate an SSH key

The key should be generated using the modern and secure ed25519 format. 

    ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C "root@onetwoseven.one"

> Please use a passphrase for ssh keys, even if it's not a good one. 

## Add to SSH agent

Make sure the agent is running. If it is, it can be added to the ssh agent: 

    ssh-add ~/.ssh/id_ed25519


