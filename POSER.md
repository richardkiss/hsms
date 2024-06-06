# Using Poser

The `poser_gen` and `poser_verify` tools can be used to prove possession of
a secret key.

This is a bit of a mess as of this writing.

Generate a secret key:

`hsmgen > sk.b32`

Create a proof of possession signing request.

`poser_gen $(hsmpk $(cat sk.b32)) i-own-this-key`

This produces a lot of output, including a `qrint` (which looks like a very
very long decimal number). Copy the coin id.

`echo ~paste-coin-id~ > coin_id.hex`

Then feed the `qrint` signature to `hsms`.

`hsms -y sk.b32`

then copy/paste the qrint decimal. Another qrint decimal will be output.

### Convert signature to hex

These steps are ugly for now, sorry about that.

Copy to a qrint file.

`echo ~paste-qrint-signature~ > sig.qr`

Convert the qrint to binary

`qrint sig.qr`

Convert the binary to hex

`python3 -c 'print(open("sig.qr.bin", "rb").read().hex())' > sig.hex`

### Verify the signature

`poser_verify $(hsmpk $(cat sk.b32)) i-own-this-key $(cat coin_id.hex) $(cat sig.hex)`

It should say `message signed correctly`.


## Why do this?

These tools are used to prevent subtraction attacks, where two (or more)
people agree to generate public keys and add them together to create a multi-sig
key. The attacker goes last. The attacker generates a key K and subtracts the
other key S (or the sum of the others keys if there is more than one other person)
from it so the final sum is a public key the attacker knows. The attacker
claims that his key is K-S, so the sum is K, which the attacker controls.
