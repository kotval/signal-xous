# Signal-Xous App

A work-in-progress towards a [Signal](https://signal.org/) client for [xous](https://github.com/betrusted-io/xous-core#xous-core).


## Working Notes

The basic idea is to save all msg directly to the PDDB, and for the UI to traverse a subset of the msg's in the PDDB.

### Next Steps

- [x] develop xous [tls library](https://github.com/betrusted-io/xous-core/pull/394) to manage the explicit trust of CA Certificates
- [x] confirm tls connection Precursor <=> chat.signal..org [ xous shellchat `wlan on` `net tls probe chat.signal.org` `net tls test chat.signal.org` ]
- [ ] develop xous websocket library to manage the setup, maintainance, and tear-dwn of a websocket connection. 
- [ ] confirm websocket connection Precursor <=> chat.signal.org
- [x] develop [xous chat library](https://github.com/betrusted-io/xous-core/pull/408) to provide a UI to interact with a Dialogue of Posts stored in the PDDB
- [ ] resurvey Whisperfish, signal-cli & gurk for useful stuff (see useful below)
- [ ] register a Precursor as a Signal device
- [ ] Maintain a fork of [libsignal](https://github.com/signalapp/libsignal)
  which renames the curve25519-dalek fork so that libsignal gets built with two
  copies of the curve25519-dalek crate, until the [lizard](https://github.com/signalapp/libsignal/issues/540#issuecomment-1779701014) issue is delt with. This is necessary to build libsignal-zkgroup 
  [ ] Consider helping upstream [lizard](https://github.com/signalapp/libsignal/issues/540#issuecomment-1779701014).
  [x] Demonstate use of libsignal-protocol to encrypt and decrypt a message in a test crate on renode
  [ ] Do the same on hardware, see if using the two different dalek crates produces a noticible lab on the non accelerated version.
  [ ] Demonstate use of libsignal-signal-crypto
  [ ] Demonstate use of libsignal-device-transfer
  [ ] Demonstate use of libsignal-zkgroup
  [ ] Demonstate use of libsignal-usernames
  [ ] Read through libsignal-media and libsignal-pin. Likely we won't need them.

### Useful

* [libsignal](https://github.com/signalapp/libsignal#overview) contains platform-agnostic APIs used by the official Signal clients and servers, written in Rust, and released under [AGPL 3.0](https://github.com/signalapp/libsignal/blob/main/LICENSE).
* The [Whisperfish](https://github.com/whisperfish/whisperfish#whisperfish) project is a helpful working example of a Signal client developed in Rust on the official Signal libsignal API. Unfortunately Whisperfish has a `tokio` dependency. See also [libsignal-service-rs](https://github.com/whisperfish/libsignal-service-rs#libsignal-service-rs) and [Presage](https://github.com/whisperfish/presage#presage)
* There is also [Signal-Desktop](https://github.com/signalapp/Signal-Desktop#signal-desktop) [signal-cli](https://github.com/AsamK/signal-cli#signal-cli) and [gurk](https://github.com/boxdot/gurk-rs#gurk-)
* A basic xous [websocket](https://github.com/nworbnhoj/xous-core/tree/websocket-lib/libs/websocket#readme) with tls support is available to connect with Signal servers.
* xous supports [qr-codes](https://github.com/betrusted-io/xous-core/blob/08aac2c2854dc3cfa7c277ddce85c0b88c72378b/services/modals/src/tests.rs#L160-L168) to facilitate device registration
* xous supports [png image processing](https://github.com/betrusted-io/xous-core/pull/207) but the websocket would need to be extended to present as a `Reader`.


### Unknowns

#### libc dependencies

xous has only limited libc support

Running `cargo tree -e no-build -e no-dev` over `libsignal` and extracted the libc dependencies gives

```
device-transfer v0.1.0 (/libsignal/rust/device-transfer)
├── boring v2.1.0 (https://github.com/signalapp/boring?branch=libsignal#3809a7e1)
│   └── libc v0.2.144
└── libc v0.2.144

libsignal-protocol v0.1.0 (/libsignal/rust/protocol)
├── curve25519-dalek v3.2.1 (https://github.com/signalapp/curve25519-dalek?branch=lizard2#4f0aa665)
│   ├── rand_core v0.5.1
│   │   └── getrandom v0.1.16
│   │       └── libc v0.2.144
├── pqcrypto-kyber v0.7.6
│   ├── libc v0.2.144
│   ├── pqcrypto-internals v0.2.4
│   │   ├── getrandom v0.2.9
│   │   │   └── libc v0.2.144
│   │   └── libc v0.2.144
├── rand v0.7.3
│   ├── libc v0.2.144

```
* [device transfer](https://github.com/signalapp/libsignal/blob/main/rust/device-transfer/src/lib.rs) can probably be re-implemented without too much fuss
* [curve25519-dalek](https://crates.io/crates/curve25519-dalek) is confirmed to work in renode.
* [pqcrypto-kyber](https://github.com/signalapp/libsignal/blob/af7bb8567c812aa13625fc90076bf71a59d64ff5/rust/protocol/src/kem.rs#L426-L455) appears to be used only in tests
* [rand]() appears to work in renode


#### 64 bit dependencies

Origionally, we were worried about these issues:

* [re-export curve25519-dalek features #453](https://github.com/signalapp/libsignal/issues/453)

* [Default Dalek32 backend on unknown platform #509](https://github.com/dalek-cryptography/curve25519-dalek/pull/509)

It seems [that](https://github.com/signalapp/libsignal/issues/453#issuecomment-1783367959) the new version 4 uses `cfg` flags so we don't have to ask them to change anything to make the necessary changes. The create builds now, but we should be careful that we are using the most performant backend with the `cfg` options.


