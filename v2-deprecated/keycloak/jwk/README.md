Testing Keys
================

These public/private keys are only for testing and should not be used in production!

To Generate New Key Pair
----------------

```bash
ssh-keygen -t rsa -b 2048 -m PEM -f jwtRS256.key
# Don't add passphrase
openssl rsa -in jwtRS256.key -pubout -outform PEM -out jwtRS256.key.pub
cat jwtRS256.key
cat jwtRS256.key.pub
```
