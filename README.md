Tree Shakeable TweetNaCl.js
============

This is a fork of [TweetNaCl.js](https://github.com/dchest/tweetnacl-js) modified to be tree shakeable, in other words, it was modified to use ES modules and exports were flattened instead of just exporting the CommonJS export as a default export (like [tweetnacl-es6](https://github.com/hakanols/tweetnacl-es6));

Exports were flattened by replacing dots with underlines. Example:

```javascript
// Instead of
const nacl = require('tweetnacl')
const exampleKeyPair = nacl.box.keyPair()

// Write the following
import { box_keyPair } from './nacl-fast.js'
const exampleKeyPair = box_keyPair()
```

Low level functions were exported directly in root level so instead of importing those using `lowlevel_` prefix you can import those directly, like this: `import { crypto_box } from './nacl-fast.js'`. Only nacl-fast.js as modified. Minified files aren't provided as this fork is made to be used with a tool that does tree shaking (like Rollup). This README.md file was simplified to only include parts that were changed by the fork, for complete info about TweetNaCl.js check the original project.

Documentation
=============

* [Build sizes](#build-sizes)
* [Usage](#usage)
  * [Public-key authenticated encryption (box)](#public-key-authenticated-encryption-box)
  * [Secret-key authenticated encryption (secretbox)](#secret-key-authenticated-encryption-secretbox)
  * [Scalar multiplication](#scalar-multiplication)
  * [Signatures](#signatures)
  * [Hashing](#hashing)
  * [Random bytes generation](#random-bytes-generation)
  * [Constant-time comparison](#constant-time-comparison)


Build sizes
-----

```javascript
import { hash } from 'tweetnacl'
console.log(hash(new Uint8Array()))
```

Build sizes after compiling the above code using Rollup then minifing:

```
FILE         GZIP     BROTLI
original.js  10.74KB  8.25KB
fork.js      2.90KB   2.37KB
```

(This section needs to be expanded in future.)


Usage
-----

### Public-key authenticated encryption (box)

Implements *x25519-xsalsa20-poly1305*.

#### box_keyPair()

Generates a new random key pair for box and returns it as an object with
`publicKey` and `secretKey` members:

    {
       publicKey: ...,  // Uint8Array with 32-byte public key
       secretKey: ...   // Uint8Array with 32-byte secret key
    }


#### box_keyPair_fromSecretKey(secretKey)

Returns a key pair for box with public key corresponding to the given secret
key.

#### box(message, nonce, theirPublicKey, mySecretKey)

Encrypts and authenticates message using peer's public key, our secret key, and
the given nonce, which must be unique for each distinct message for a key pair.

Returns an encrypted and authenticated message, which is
`box_overheadLength` longer than the original message.

#### box_open(box, nonce, theirPublicKey, mySecretKey)

Authenticates and decrypts the given box with peer's public key, our secret
key, and the given nonce.

Returns the original message, or `null` if authentication fails.

#### box_before(theirPublicKey, mySecretKey)

Returns a precomputed shared key which can be used in `box_after` and
`box_open_after`.

#### box_after(message, nonce, sharedKey)

Same as `box`, but uses a shared key precomputed with `box_before`.

#### box_open_after(box, nonce, sharedKey)

Same as `box_open`, but uses a shared key precomputed with `box_before`.

#### Constants

##### box_publicKeyLength = 32

Length of public key in bytes.

##### box_secretKeyLength = 32

Length of secret key in bytes.

##### box_sharedKeyLength = 32

Length of precomputed shared key in bytes.

##### box_nonceLength = 24

Length of nonce in bytes.

##### box_overheadLength = 16

Length of overhead added to box compared to original message.


### Secret-key authenticated encryption (secretbox)

Implements *xsalsa20-poly1305*.

#### secretbox(message, nonce, key)

Encrypts and authenticates message using the key and the nonce. The nonce must
be unique for each distinct message for this key.

Returns an encrypted and authenticated message, which is
`secretbox_overheadLength` longer than the original message.

#### secretbox_open(box, nonce, key)

Authenticates and decrypts the given secret box using the key and the nonce.

Returns the original message, or `null` if authentication fails.

#### Constants

##### secretbox_keyLength = 32

Length of key in bytes.

##### secretbox_nonceLength = 24

Length of nonce in bytes.

##### secretbox_overheadLength = 16

Length of overhead added to secret box compared to original message.


### Scalar multiplication

Implements *x25519*.

#### scalarMult(n, p)

Multiplies an integer `n` by a group element `p` and returns the resulting
group element.

#### scalarMult_base(n)

Multiplies an integer `n` by a standard group element and returns the resulting
group element.

#### Constants

##### scalarMult_scalarLength = 32

Length of scalar in bytes.

##### scalarMult_groupElementLength = 32

Length of group element in bytes.


### Signatures

Implements [ed25519](http://ed25519.cr.yp.to).

#### sign_keyPair()

Generates new random key pair for signing and returns it as an object with
`publicKey` and `secretKey` members:

    {
       publicKey: ...,  // Uint8Array with 32-byte public key
       secretKey: ...   // Uint8Array with 64-byte secret key
    }

#### sign_keyPair_fromSecretKey(secretKey)

Returns a signing key pair with public key corresponding to the given
64-byte secret key. The secret key must have been generated by
`sign_keyPair` or `sign_keyPair_fromSeed`.

#### sign_keyPair_fromSeed(seed)

Returns a new signing key pair generated deterministically from a 32-byte seed.
The seed must contain enough entropy to be secure. This method is not
recommended for general use: instead, use `sign_keyPair` to generate a new
key pair from a random seed.

#### sign(message, secretKey)

Signs the message using the secret key and returns a signed message.

#### sign_open(signedMessage, publicKey)

Verifies the signed message and returns the message without signature.

Returns `null` if verification failed.

#### sign_detached(message, secretKey)

Signs the message using the secret key and returns a signature.

#### sign_detached_verify(message, signature, publicKey)

Verifies the signature for the message and returns `true` if verification
succeeded or `false` if it failed.

#### Constants

##### sign_publicKeyLength = 32

Length of signing public key in bytes.

##### sign_secretKeyLength = 64

Length of signing secret key in bytes.

##### sign_seedLength = 32

Length of seed for `sign_keyPair_fromSeed` in bytes.

##### sign_signatureLength = 64

Length of signature in bytes.


### Hashing

Implements *SHA-512*.

#### hash(message)

Returns SHA-512 hash of the message.

#### Constants

##### hash_hashLength = 64

Length of hash in bytes.


### Random bytes generation

#### randomBytes(length)

Returns a `Uint8Array` of the given length containing random bytes of
cryptographic quality.

**Implementation note**

TweetNaCl.js uses the following methods to generate random bytes,
depending on the platform it runs on:

* `window.crypto.getRandomValues` (WebCrypto standard)
* `window.msCrypto.getRandomValues` (Internet Explorer 11)
* `crypto.randomBytes` (Node.js)

If the platform doesn't provide a suitable PRNG, the following functions,
which require random numbers, will throw exception:

* `randomBytes`
* `box_keyPair`
* `sign_keyPair`

Other functions are deterministic and will continue working.

If a platform you are targeting doesn't implement secure random number
generator, but you somehow have a cryptographically-strong source of entropy
(not `Math.random`!), and you know what you are doing, you can plug it into
TweetNaCl.js like this:

    setPRNG(function(x, n) {
      // ... copy n random bytes into x ...
    });

Note that `setPRNG` *completely replaces* internal random byte generator
with the one provided.


### Constant-time comparison

#### verify(x, y)

Compares `x` and `y` in constant time and returns `true` if their lengths are
non-zero and equal, and their contents are equal.

Returns `false` if either of the arguments has zero length, or arguments have
different lengths, or their contents differ.

