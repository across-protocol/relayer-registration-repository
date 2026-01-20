# Relayer Registration

This repository allows for third party relayers to register with the Across nomination system. The
Across nomination system is an algorithm that assigns particular a (short) period of exclusivity to
relayers based on a mixture of their pricing and their performance.

This registration comes with no guarantees that your relayer will be assigned nominated deposits
and, by submitting a configuration here, you agree that all assignments are at the sole discretion
of Risk Labs.


## Submitting a configuration

### Step 1: Create your registration file

Create a file titled `<identifier>.json` in the `relayer_registration/` directory, where
`<identifier>` is some string that you would like to be associated with your relayer.

For example, you might choose to have your file be called something like `risklabs.json` and place
it at `relayer_registration/risklabs.json`.

The file should have the following keys:

* `public_key`: A hex-encoded Ed25519 public key (64 characters) used to authenticate requests from
  your relayer. You will sign configuration updates with the corresponding private key to prove
  ownership.
* `exclusivity_address`: This is the default address that exclusivity is assigned to. Used as
  fallback for any chain not specified in `exclusivity_addresses`.
* `exclusivity_addresses`: This allows you to override the exclusivity address on a per-chain basis.
  The key is the chain ID (as a string), and the value is the address for that chain. This supports
  both EVM addresses and non-EVM addresses (e.g., Solana). When you are nominated for a transfer,
  the system will use the chain-specific address if provided, otherwise it falls back to
  `exclusivity_address`.
* `active`: This is a boolean that denotes whether your relayer is currently active and eligible for
  nomination.

Here is an example of what the file should look like

```json
{
  "public_key": "c3e1d9fd756ee7bd8c50d416c1f4aec8d47f49b88ede147769f90dad45de0d68",
  "exclusivity_address": "0x07aE8551Be970cB1cCa11Dd7a11F47Ae82e70E67",
  "exclusivity_addresses": {
    "999": "0xfc0a7211882391717cEdF446889Af99Ca4168093",
    "34268394551451": "FmMK62wrtWVb5SVoTZftSCGw3nEDA79hDbZNTRnC1R6t"
  },
  "active": true
}
```

### Step 2: Submit your registration via Pull Request

Once you've created your registration file:

1. **Fork this repository** to your GitHub account
2. **Create a new branch** for your registration (e.g., `add-risklabs-relayer`)
3. **Add your file** to the `relayer_registration/` directory
4. **Commit your changes** with a clear message (e.g., "Add RiskLabs relayer registration")
5. **Create a Pull Request** to the main branch of this repository

Your PR will be automatically validated by GitHub Actions to ensure:
* Your JSON file is properly formatted
* The `public_key` is exactly 64 hex characters (valid Ed25519 public key)
* The `exclusivity_address` is a valid Ethereum address (0x + 40 hex characters)
* All addresses in `exclusivity_addresses` are valid for their respective chains
* The `active` field is a boolean (true or false)

If validation fails, the PR will show an error and you'll need to fix your file before it can be
merged.

### Step 3: Wait for review and merge

Once your PR passes validation, a maintainer will review your registration. After approval and
merge:

1. Your registration will be **automatically synced to the registration database** (typically within
   1 second via GitHub webhook)
2. You will be able to **authenticate with the Configuration API** using your private key to submit
   your pricing and balance information
3. Your relayer will be **eligible for nomination** once you've submitted a valid configuration to
   the Configuration API


## Updating your real-time configuration

Once the pull request to add your relayer's address to this repository is added, you must interact
with the real-time configuration API which allows you to specify information that allows us to
assign deposits that fit your desired criteria.

### Configurations

Relayers are expected to submit several relevant objects:

#### `tokens`

The `tokens` object contains information about which tokens a relayer considers equivalent and
plans to price the same.

It has the following structure:

```
{
  "eth": [
    { "address": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2", "chainId": 1, "decimals": 18 },
    { "address": "0x4200000000000000000000000000000000000006", "chainId": 8453, "decimals": 18 }
  ],
  "stable": [
    { "address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48", "chainId": 1, "decimals": 6 },
    { "address": "0xdAC17F958D2ee523a2206206994597C13D831ec7", "chainId": 1, "decimals": 6 },
    { "address": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913", "chainId": 8453, "decimals": 6 },
    { "address": "0xfde4C96c8593536E31F229EA8f37b2ADa2699bb2", "chainId": 8453, "decimals": 6 }
  ]
}
```

#### `balances`

The `balances` object contains information about the available funds for each relayer that they
can use for fulfilling orders.

Relayers are expected to keep this information accurate and up-to-date. Balances are represented
as strings to support large integer values (BigInt) and should be reported in wei.

```
{
  "1": {
    "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2": "10000000000000000000",
    "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48": "1000000000"
  },
  "8453": {
    "0x4200000000000000000000000000000000000006": "5000000000000000000",
    "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913": "500000000"
  }
}
```

#### `orderbook`

The `orderbook` object contains information about the bid-ask for each token that the relayer
supports bridging for. This allows the relayer to specify how much they value a token on certain
chains relative to others.

The key of this object is the string used to represent token groups in the `token` object and a
relayer basically specifies a sequence of bids/asks for each chain (these are interpreted
cumulatively).

See example structure below:

```
{
  "eth": {
    "bid": {
      "1": [
        { "amount": 3, "price": 0.999 },
        { "amount": 7, "price": 0.99 },
        { "amount": 5, "price": 0.98 }
      ],
      "8453": [
        { "amount": 3, "price": 0.999 },
        { "amount": 7, "price": 0.99 },
        { "amount": 5, "price": 0.98 }
      ]
    },
    "ask": {
      "1": [
        { "amount": 5, "price": 1.001 },
        { "amount": 10, "price": 1.01 }
      ],
      "8453": [
        { "amount": 2, "price": 1.001 },
        { "amount": 7, "price": 1.01 },
        { "amount": 6, "price": 1.02 }
      ]
    }
  },
  "stable": {
    "bid": {
      "1": [
        { "amount": 500, "price": 0.9995 },
        { "amount": 1000, "price": 0.999 }
      ]
    },
    "ask": {
      "1": [
        { "amount": 500, "price": 1.001 },
        { "amount": 500, "price": 1.005 },
        { "amount": 500, "price": 1.01 }
      ],
      "8453": [
        { "amount": 1000, "price": 1.001 },
        { "amount": 1000, "price": 1.005 }
      ]
    }
  }
}
```

#### `exchangerate`

The exchange rate key allows relayers to specify an exchange rate between the different groups
defined in the `tokens` object.

There is an implicit 1:1 exchange rate for anything within the `tokens` group -- i.e. if USDC on
Base and USDT on Ethereum are in the same group then we only determine the price between those two
from the `pricing` object since we assume a 1:1 exchange rate.

Each exchange rate should have an expiration expressed as a unix timestamp. The maximum validity
of this expiration should be 1 hour and the minimum validity should be 10 seconds. If you
deliver values outside of this range then it will be rejected.

The exchange rate will be assumed to be valid until it is either replaced by a new value OR
reaches the expiration timestamp. This means that you can submit exchange rate information in
bundles with multiple exchange rates or one at a time.

```
{
  "eth": {
    "stable": { "numerator": 1, "denominator": 3150, "expiration": 1767900109 }
  },
  "stable": {
    "eth": { "numerator": 3200, "denominator": 1, "expiration": 1767900469 }
  }
}
```

### Authentication

**All endpoints require the `X-Relayer-Id` header** to identify which relayer's configuration to
access.

**Authenticated endpoints** (all except `/api/checkLive`) additionally require signature
verification to ensure that a relayer's configuration can only be read or written by the relayer
themself.

The relayer's public key is stored publicly in the relayer registration repository and relayers
must keep their private key secret, or non-authorized individuals will be able to read or write to
your configuration.

**Required headers for authenticated endpoints:**

```
X-Relayer-Id: <String used to identify your configuration file>
X-Timestamp: <Unix timestamp (ms) when you submitted -- Will error if older than 5 minutes>
X-Signature: <hex-encoded-64-byte-signature>
```

**For `/api/checkLive`**: Only `X-Relayer-Id` is required (no signature verification).

**Authorization**: Relayers can only access their own configuration. You cannot read or modify
another relayer's configuration, but you can check if any relayer is active via `/api/checkLive`.

The `X-Signature` is created by building a specific message. This message is defined as:

```
message = timestamp + "\n" + HTTP method + "\n" + API endpoint + "\n" + body_hash
```

You can then take this message and sign it with your ED25519 private key. The server will then
lookup the relevant public key using `X-Relayer-Id` and reconstruct the message and verify that the
signature is valid with the relayers public key.

You can find instructions below for how to construct such a key.

### Pricing examples

These two pricing examples assume that an input amount is specified but we can go in the reverse
order of calculations if an output amount is specified

#### Example 1: ETH-ETH

Suppose a user was transferring 10 WETH from Ethereum to Base and that relayer `0x1` had a
configuration identical to the examples from [the objects section](objects). Then `0x1` would
demand 9.87273 WETH as an output for the input of 10 WETH.

9.87273 can be computed from the bid and asks:

* The relayer would take the first 3 WETH on Ethereum at 0.999 and the next 7 at 0.99 which
  results in a bid fee of `(3*(1 - 0.999) + 7*(1 - 0.99)) = 0.073` ETH
* The relayer would then be willing to provide the first 5 WETH on Base at price of 1.001 and the
  next 4.927 at 1.01 for a fee of `(5*(1.001 - 1) + 4.927*(1.01 - 1)) = 0.05427`

This gives the total fee of 0.12727.

#### Example 2: STABLE-ETH

Suppose a user was transferring 1,500 USDC from Ethereum to Base ETH and that relayer `0x1` had
a configuration identical to the examples from [the objects section](objects). Then `0x1` would
demand 0.467891015625 as an output for the input of 1,500 USDC.

X can be computed from the bid and asks:

* The relayer would take the first 500 on Ethereum at 0.9995 and the next 1,000 at 0.999 which
  results in a bid fee of `(500*(1 - 0.9995) + 1000*(1 - 0.999)) = 1.25` USDC
* A total of `1,498.75` USDC would then be exchanged to 0.468359375 ETH according to the
  `stable->eth` exchange (`3200/1 * 1/1498.75`)
* The relayer would be willing to provide the entire amount at a price of 1.001 which leads to
  a fee of `(0.468359375*(1.001 - 1)) = 0.000468359375`
* The output amount would then be 0.467891015625 (`0.468359375 - 0.000468359375`)


### Endpoints

There are several key endpoints in the API. All endpoints use header-based identification (no query
parameters). The `X-Relayer-Id` header is required for all endpoints.

For authenticated endpoints, you must also include `X-Timestamp` and `X-Signature` headers. For
`/api/checkLive`, only `X-Relayer-Id` is required - signature headers are optional and will be
ignored if provided.

* `api/checkLive` (`GET`, unauthenticated): Responds with whether a relayer is currently
  active (based on whether their configuration is set `active: true` or `active: false`).
* `api/getConfiguration` (`GET`, authenticated): Fetches a relayer's full configuration
* `api/getBalance` (`GET`, authenticated): Fetches a relayer's current balance data
* `api/updateBalance` (`POST`, authenticated): Allows a relayer to update their token balances. Data
  goes stale after 30 minutes.
* `api/getExchangeRate` (`GET`, authenticated): Fetches a relayer's current exchange rates and
  expiry information
* `api/updateExchangeRate` (`POST`, authenticated): Allows a relayer to update their posted exchange
  rates and set expiries -- A new exchange rate should deprecate any previously set exchange rates
  but we recommend using a short expiration window (<30 seconds) to ensure prices expire quickly
* `api/getOrderBook` (`GET`, authenticated): Fetches a relayer's current orderbook information
* `api/updateOrderBook` (`POST`, authenticated): Allows a relayer to update their order book
  information
* `api/getToken` (`GET`, authenticated): Fetches a relayer's current token configuration
* `api/updateToken` (`POST`, authenticated): Allows a relayer to update their token lists. Data
  persists indefinitely.


## Instructions for generating a public key

Your Ed25519 keypair consists of a private key (keep this secret!) and a public key (submit this in
your registration). Ed25519 is a digital signature algorithm—you will use your private key to sign
requests, and the server will verify the signature using your registered public key.

> **⚠️ Important**: Store your private key securely. You will need it to sign configuration updates
  sent to the configuration endpoint. _Never share your private key or commit it to version control._

Below are some examples for generating a keypair in various languages and via command line.

### Command Line (using OpenSSL)

Generate a keypair and extract the public key in hex format:

```bash
# Generate a private key
openssl genpkey -algorithm Ed25519 -out private_key.pem

# Extract the raw private key (32 bytes) as hex
openssl pkey -in private_key.pem -text -noout 2>/dev/null | grep -A 2 "priv:" | tail -n 2 | tr -d ' \n:'

# Extract the raw public key (32 bytes) as hex
openssl pkey -in private_key.pem -pubout -outform DER | tail -c 32 | xxd -p -c 32
```

Or as a one-liner that outputs both keys:

```bash
openssl genpkey -algorithm Ed25519 -out private_key.pem && \
echo "Private Key: $(openssl pkey -in private_key.pem -text -noout 2>/dev/null | grep -A 2 'priv:' | tail -n 2 | tr -d ' \n:')" && \
echo "Public Key: $(openssl pkey -in private_key.pem -pubout -outform DER | tail -c 32 | xxd -p -c 32)"
```

### TypeScript

Install the `tweetnacl` package:

```bash
npm install tweetnacl
```

```typescript
import nacl from 'tweetnacl';

// Generate a new Ed25519 signing keypair
const keypair = nacl.sign.keyPair();

// Convert to hex strings
// Note: secretKey is 64 bytes (32-byte seed + 32-byte public key), we only need the first 32 bytes
const publicKeyHex = Buffer.from(keypair.publicKey).toString('hex');
const privateKeyHex = Buffer.from(keypair.secretKey.slice(0, 32)).toString('hex');

console.log('Public Key (submit this):', publicKeyHex);
console.log('Private Key (keep secret!):', privateKeyHex);
```

### Python

Install the `PyNaCl` package:

```bash
pip install pynacl
```

```python
from nacl.signing import SigningKey

# Generate a new Ed25519 signing keypair
signing_key = SigningKey.generate()
verify_key = signing_key.verify_key

# Convert to hex strings
public_key_hex = verify_key.encode().hex()
private_key_hex = signing_key.encode().hex()

print(f"Public Key (submit this): {public_key_hex}")
print(f"Private Key (keep secret!): {private_key_hex}")
```

### Rust

Add the following to your `Cargo.toml`:

```toml
[dependencies]
ed25519-dalek = "2"
rand = "0.8"
hex = "0.4"
```

```rust
use ed25519_dalek::SigningKey;
use rand::rngs::OsRng;

fn main() {
    // Generate a new Ed25519 signing keypair
    let signing_key = SigningKey::generate(&mut OsRng);
    let verifying_key = signing_key.verifying_key();

    // Convert to hex strings
    let public_key_hex = hex::encode(verifying_key.as_bytes());
    let private_key_hex = hex::encode(signing_key.to_bytes());

    println!("Public Key (submit this): {}", public_key_hex);
    println!("Private Key (keep secret!): {}", private_key_hex);
}
```
