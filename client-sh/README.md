# nry.testpoint.io test client

Bunch of shell scripts for simple communication with NRY API.

Note: if you create var directory here it will be ignored from the git.

## Usage

You can skip JWT steps if only public read-only endpoints are going to be used.

1. Get your JWT token for nry.testpoint.io at idp.testpoint.io
2. `export NRY_AUTH="JWT {rendered token from the idp.testpoint.io, you can get it as synthetic user}"`
3. copy some .sample scripts to .sh versions or use them in place
4. try `do_demo_auth` script to make sure auth works.

## HOC Archive validation example

Example of downloading and verifying some HOC archive. Commands using local gpg2 installation to avoid keys interfere with your main one, and demonstrate how it works without side-effects.

Let's assume we are validating archive with IPFS hash `QmS5EogHn2MtSBsQ4wXFEPeiA1zWPfaEbumWEJJNsUWuW2`.

    # just for shortness
    $ alias localgpg="gpg2 --homedir gpghome"

    $ mkdir test_gpg && cd test_gpg && mkdir gpghome && chmod 0700 gpghome

    # list public keys
    $ localgpg -k
    # list private keys
    $ localgpg -K
    (expect nothing in both cases except message about database created for the first time)

    # download files (you can use IPFS, not HTTP gateway, if you want)
    $ wget https://ipfs.io/ipfs/QmS5EogHn2MtSBsQ4wXFEPeiA1zWPfaEbumWEJJNsUWuW2/proof.json
    $ wget https://ipfs.io/ipfs/QmS5EogHn2MtSBsQ4wXFEPeiA1zWPfaEbumWEJJNsUWuW2/proof.sig

    # Try to verify signature:
    $ localgpg --verify proof.sig proof.json
    gpg: Signature made Mon 13 Feb 2017 14:28:26 +04 using RSA key ID 43DF36570DC58985
    gpg: Can't check signature: No public key

    # extract public key from proof.json file
    $ python -c "import json; open('proof.pubkey', 'wb').write(json.loads(open('proof.json').read())['pub_key'])"
    $ cat proof.pubkey
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Version: GnuPG v1
    ...
    (expect public key armor representation)
    ...
    -----END PGP PUBLIC KEY BLOCK-----

    $ localgpg --import proof.pubkey
    gpg: key 43DF36570DC58985: public key "nry.testpoint.io (ausdigital-nry/1) <hi@testpoint.io>" imported
    gpg: Total number processed: 1
    gpg:               imported: 1

    $ localgpg -k
    ..../client-sh/var/test_gpg/gpghome/pubring.kbx
    -----------------------------------------------
    pub   rsa1024 2017-02-10 [SC] [expires: 2022-02-09]
          2CF52FFAAD9225CAFBB5AB5E43DF36570DC58985
    uid           [ unknown] nry.testpoint.io (ausdigital-nry/1) <hi@testpoint.io>

    $ localgpg -K
    (empty output - no private keys imported)

    $ localgpg --verify proof.sig proof.json
    gpg: Signature made Mon 13 Feb 2017 14:28:26 +04 using RSA key ID 43DF36570DC58985
    gpg: Good signature from "nry.testpoint.io (ausdigital-nry/1) <hi@testpoint.io>" [unknown]
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: 2CF5 2FFA AD92 25CA FBB5  AB5E 43DF 3657 0DC5 8985

If you remove gpg key (`localgpg --delete-key 2CF52FFAAD9225CAFBB5AB5E43DF36570DC58985`) verification step will fail again.

Please note warning about untursted signature. It's okay, because gpg2 client doesn't have it in any trust database. You can manually trust it or validate it by different ways (ensure your DCP have this key in list of the keys for given business, notary in this case).
