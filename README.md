# luce-keychain

Secrets in the system's own credential store, for
[luce-base](https://github.com/dymokomi/luce-base) and Luce programs:

| System | Store |
| --- | --- |
| macOS | the user's Keychain: generic-password items (Security.framework) |
| Windows | the Credential Manager: generic credentials named `service/account` (advapi32) |
| Linux | the Secret Service through libsecret, loaded at run time with `dlopen`; nothing is linked |

```luce
from luce_keychain import keychain

keychain.store("com.example.mail", "alice@example.com", password)
let password = keychain.load("com.example.mail", "alice@example.com")   # fails with not_found when absent
if keychain.contains("com.example.mail", "bob@example.com"): …
keychain.remove("com.example.mail", "alice@example.com")                # nothing kept is not an error
```

A secret is text of at most 2560 bytes (the Windows limit), kept under a service name
and an account, each 1–256 bytes without control characters. Storing replaces whatever
was kept for the pair. Errors: `not_found`, `unavailable` (no reachable store, such as
a Linux system without libsecret or a running Secret Service), `failed` (the store
refused) and `invalid` (bad names or secret).

From Base, `load` returns an `interop.Owned[str]`; release it when done.

## Test

```sh
./test.sh
```

The tests make a round trip through the real store, under the service
`com.luciaos.luce-keychain.test`, and clean up after themselves. A system with no
reachable store says so and checks argument validation only.

## License

Dual-licensed under Apache-2.0 or MIT, at your option.
