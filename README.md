# Relayer Registration

This repository allows for third party relayers to register with the Across nomination system. The
Across nomination system is an algorithm that assigns particular a (short) period of exclusivity to
relayers based on a mixture of their pricing and their performance.

This registration comes with no guarantees that your relayer will be assigned nominated deposits
and, by submitting a configuration here, you agree that all assignments are at the sole discretion
of Risk Labs.

## Submitting a configuration

In order to submit a configuration you should create a file titled with `<identifier>.json` where
`<identifier>` is some string that you would like to be associated with your relayer.

For example, you might choose to have your file be called something like `risklabs.json`.

The file should have the following keys:

* `public_key`: A hex-encoded X25519 public key (64 characters) used to encrypt your relayer
  configuration.
* `exclusivity_address`: This is the default address that exclusivity is assigned to on EVM chains.
* `exclusivity_addresses`: This allows you to override the address that is assigned exclusivity both
  on EVM chains and allows you to give a non-EVM address.
* `active`: This is a boolean that simply denotes whether or not your relayer is currently active.

### Example file

```json
{
  "public_key": "c3e1d9fd756ee7bd8c50d416c1f4aec8d47f49b88ede147769f90dad45de0d68",
  "exclusivity_address": "0x07aE8551Be970cB1cCa11Dd7a11F47Ae82e70E67",
  "exclusivity_addresses": {
    999: "0xfc0a7211882391717cEdF446889Af99Ca4168093",
    34268394551451: "FmMK62wrtWVb5SVoTZftSCGw3nEDA79hDbZNTRnC1R6t"
  },
  "active": true
}
```

### Generating a public key

Your X25519 keypair consists of a private key (keep this secret!) and a public key (submit this in
your registration).

> **⚠️ Important**: Store your private key securely. You will need it to encrypt configuration data
  that you send to the configuration endpoint. _Never share your private key or commit it to
  version control._

Below are examples for generating a keypair in various languages and via command line.

#### Command Line (using OpenSSL)

Generate a keypair and extract the public key in hex format:

```bash
# Generate a private key
openssl genpkey -algorithm X25519 -out private_key.pem

# Extract the raw private key (32 bytes) as hex
openssl pkey -in private_key.pem -text -noout 2>/dev/null | grep -A 2 "priv:" | tail -n 2 | tr -d ' \n:'
echo  # newline

# Extract the raw public key (32 bytes) as hex
openssl pkey -in private_key.pem -pubout -outform DER | tail -c 32 | xxd -p -c 32
```

Or as a one-liner that outputs both keys:

```bash
openssl genpkey -algorithm X25519 -out private_key.pem && \
echo "Private Key: $(openssl pkey -in private_key.pem -text -noout 2>/dev/null | grep -A 2 'priv:' | tail -n 2 | tr -d ' \n:')" && \
echo "Public Key: $(openssl pkey -in private_key.pem -pubout -outform DER | tail -c 32 | xxd -p -c 32)"
```

#### TypeScript

Install the `tweetnacl` package:

```bash
npm install tweetnacl
```

```typescript
import nacl from 'tweetnacl';

// Generate a new X25519 keypair
const keypair = nacl.box.keyPair();

// Convert to hex strings
const publicKeyHex = Buffer.from(keypair.publicKey).toString('hex');
const privateKeyHex = Buffer.from(keypair.secretKey).toString('hex');

console.log('Public Key (submit this):', publicKeyHex);
console.log('Private Key (keep secret!):', privateKeyHex);
```

#### Python

Install the `PyNaCl` package:

```bash
pip install pynacl
```

```python
from nacl.public import PrivateKey

# Generate a new X25519 keypair
private_key = PrivateKey.generate()
public_key = private_key.public_key

# Convert to hex strings
public_key_hex = public_key.encode().hex()
private_key_hex = private_key.encode().hex()

print(f"Public Key (submit this): {public_key_hex}")
print(f"Private Key (keep secret!): {private_key_hex}")
```

#### Rust

Add the following to your `Cargo.toml`:

```toml
[dependencies]
x25519-dalek = "2"
rand = "0.8"
hex = "0.4"
```

```rust
use x25519_dalek::{StaticSecret, PublicKey};
use rand::rngs::OsRng;

fn main() {
    // Generate a new X25519 keypair
    let private_key = StaticSecret::random_from_rng(OsRng);
    let public_key = PublicKey::from(&private_key);

    // Convert to hex strings
    let public_key_hex = hex::encode(public_key.as_bytes());
    let private_key_hex = hex::encode(private_key.as_bytes());

    println!("Public Key (submit this): {}", public_key_hex);
    println!("Private Key (keep secret!): {}", private_key_hex);
}
```
