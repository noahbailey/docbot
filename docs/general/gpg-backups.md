# Encrypt Backups with GPG

## Export the key

On the trusted host, [generate a keypair](/general/gpg)

    gpg --output ~/backup_key.key --armor --export admin@example.com

Copy your key to the server which will be creating backups

    scp ~/backup_key.key server.example.com:~/

## Import the key on a server

    gpg --import < ~/backup_key.key

Check that the key is present

    gpg -k

## Example: encrypting a database backup

This one-liner exports a mariadb database, then compresses it and encrypts using the GPG key: 

    sudo mysqldump my-database | gzip | gpg -r admin@example.com --encrypt -o /backups/my-database.gpg

