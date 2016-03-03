# critical_data

## Yubikey Process for GPG management

File myreset : 

```
/hex
scd serialno
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 e6 00 00
scd apdu 00 44 00 00
/echo Card has been successfully reset.
```

> gpg-connect-agent -r myreset

Starts by reseting the yubikey doing : 4 attempts with a wrong PIN (81), 4 attempts
with a wrong ADMIN PIN (83).
then terminate the card with (e6) and reactivate it with (44)

Then change the PIN (default 123456) and ADMIN PIN (default 12345678) with
`gpg2 --change-pin`. then type "admin" to have access to all options.

Then 2 options : 

- either you create a key for the first time, so 
  - create a 4096 RSA signature key and 2 2048 RSA subkeys: 1 for auth, 1 for encryption : `gpg2 --gen-key --expert` 
    and `gpg2 --expert --edit-key XXXXX`, then `addkey`, `addsubkey`.
  - Create a complex paraphrase with a mechanism to find it
  - Create a PaperKey with base64 on one side and QRCodes on the other: of the
    GPG cert but keys still encrypted with the paraphrase.
  - print several copies, then write BY HAND the mechanism to find the paraphrase on the papers
  - CAUTION: if you have time, try at least one time to make the recovery
    process to test that there is no mistake.
  - then save and publish the public certificate as much as you can, `gpg -a
    --export XXXX > pubkey.asc` for instance into __keybase__.
  - then use `gpg2 --change-pin` again, to set the *url* field inside the
    smartcard to the public url of the certificate

- or you recover a previously created key from a "Paperkey" backup, in this case:
  - scan QR-codes-> concatenate them
  - if the QR-codes are broken, use tesseract to make some OCR and retrieve the raw base64
  - retrieve the published pubkey
  - use PaperKey import base64 thepubkey theoutputprivkey
  - `gpg2 --import theoutputprivkey` and delete theoutputprivkey

At this point, we have one private key containing 3 keys (1 main for sig and 2
others for encryption and authentication) protected with a complex paraphrase.

The last thing you have to do is to transfer them to the yubikey :

```bash
gpg2 --expert --edit-key XXXXX
toggle # to switch to private keys
key 0 # select the key you want to send
keytocard # will send the key to the card, and replace the one on the PC with a
          # "stub" saying: the key is on the card ! so warning, that mean the
          # key is not readable anymore and no backup are doable
1 : key0 is the main one, attach it to slot 1 : signature on the card
key 1
keytocard
2
key 2
keytocard
3
```

One you have done that, to use the yubikey on another PC, you need to 
- first you need the software: install GPG on it
- then we need the public certificate, to do that: 
  `gpg2 --change-pin`, then call the `fetch` command, which will download and
  install from the URL we set earlier the public certificate.
- then we need to define the "stub" : ie the flag to say that the private key
  is available BUT on a smartcard you need to plug in: `gpg2 --card-status`
  will do the job.

Then GO !

