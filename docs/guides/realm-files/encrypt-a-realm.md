# Encrypt a Realm - C++ SDK
You can encrypt the realm file on disk with AES-256 +
SHA-2 by supplying a 64-byte encryption key when opening a
realm.

Realm transparently encrypts and decrypts data with standard
[AES-256 encryption](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) using the
first 256 bits of the given 512-bit encryption key. Realm
uses the other 256 bits of the 512-bit encryption key to validate
integrity using a [hash-based message authentication code
(HMAC)](https://en.wikipedia.org/wiki/HMAC).

> Warning:
> Do not use cryptographically-weak hashes for realm encryption keys.
For optimal security, we recommend generating random rather than derived
encryption keys.
>

Encrypt a realm by calling the `set_encryption_key()` function on
your db_config:

```cpp
// Check if we already have a key stored in the platform's secure storage.
// If we don't, generate a new one.
// Use your preferred method to generate a key. This example key is
// NOT representative of a secure encryption key. It only exists to
// illustrate the form your key might take.
std::array<char, 64> exampleKey = {
    0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 2, 2, 0, 0, 0, 0,
    0, 0, 3, 3, 0, 0, 0, 0, 0, 0, 4, 4, 0, 0, 0, 0, 0, 0, 5, 5, 0, 0,
    0, 0, 0, 0, 6, 6, 0, 0, 0, 0, 0, 0, 7, 7, 0, 0, 0, 0, 0, 0};

// Store the key securely to be used next time we want to open the database.
// We don't illustrate this here because it varies depending on the platform.

// Create a database configuration.
auto config = realm::db_config();
// Set the encryption key in your config.
config.set_encryption_key(exampleKey);

// Open or create a database with the config containing the encryption key.
auto realm = realm::db(config);

```

> Tip:
> The C++ SDK does not yet support encrypting a realm that already
exists on device. You must encrypt the realm the first time you open it.
>

## Store & Reuse Keys
You **must** pass the same encryption key every time you open the encrypted realm.
If you don't provide a key or specify the wrong key for an encrypted
realm, the Realm SDK throws an error.

Apps should store the encryption key securely on the device so that other
apps cannot read the key.

## Performance Impact
Reads and writes on encrypted realms can be up to 10% slower than unencrypted realms.
