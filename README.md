secretctl
---------

GPG wrapper to simplify multi-user encryption/decryption of secrets


usage
--------------
Usage can be seen by running `secretctl` with no args

```
Usage: secretctl CMD [ARGS]

A simple gpg-based workflow for shared encryption of secrets.

secretctl works by maintaining a directory containing public keys
and a keyfile that maps the key ID to the key name (basename of the key file)

Generate a key with: gpg --gen-key

Commands:

  export KEYID [KEYNAME]  Exports a gpg key to be shared for future encryptions.
                              Exports to .gpg-keys keydir searching up from working dir
                              Will create .gpg-keys in current directory if not found
                          KEYID is the hex ID of a public key which is output when
                              creating (gpg --gen-key) or listing keys (gpg --list-keys):
                              - given: pub rsa2048/A69BF163 2015-03-29
                              - KEYID is A69BF163
                          KEYNAME is a human-recognizable short name for the key
                              Defaults to anowell_olaf

  import                  Imports all public keys in the keydir into your keychain
                          This allows encryption of files that can be decrypted by others

  encrypt FILENAME ...    Encrypts one or more files with all the public keys in the keydir
                          Outputs the encrypted file(s) as FILENAME.gpg

  decrypt FILENAME ...    Decrypts one or more files with your keyring
                          Outputs the decrypted file(s) without the .gpg suffix

  clean                   Remove all decrypted files that have an encrypted counterpart
```


workflow
--------

This is the basic workflow for getting started
```
$ gpg --gen-key
...interactive key generation...
gpg: key A59B011E marked as ultimately trusted
...snip...
$ secretctl export A59B011E
Complete. Exported A59B011E as anowell_olaf and added to the keylist.
```

This is the basic workflow to encrypt and decrypt a secret:
```
$ echo "top secret stuff" > my_secret
$ secretctl encrypt my_secret
Complete. Encrypted as my_secret.gpg
$ secretctl clean
removed 'my_secret'
$ secretctl decrypt my_secret.gpg
Complete. Decrypted to my_secret
$ cat my_secret
top secret stuff
```

This is the workflow for re-encrypting a secret for an additional user to access:
```
# After another user exports a GPG key using secretctl:
$ secretctl import
Importing /path/to/gpg/anowell_olaf.pub
...snip...
Importing /path/to/gpg/johndoe_serenity.pub
...snip...
Finished importing all public keys. Ready to encrypt.
$ secretctl decrypt my_secret.gpg
Complete. Decrypted to my_secret
$ secretctl encrypt my_secret
Complete. Encrypted to my_secret.gpg
$ secretctl clean
removed 'my_secret'

# Now my_secret can be decrypted with the private key for either anowell_olaf or johndoe_serenity
```
