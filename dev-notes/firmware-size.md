# Firmware size analysis

Firmware size has been analyzed using `cargo-bloat` and latest firmware version
(from PR [#15](https://github.com/fobnail/fobnail/pull/15)). Latest firmware
does not build anymore (with default flags) on nRF target due code overflowing
nRF flash size, to workaround this (`cargo-bloat` requires a built binary to
analyze) increase flash size in `external/nrf-hal/nrf52840-hal/memory.x`.

## Unoptimized binary size

Test below shows analysis of unoptimized code (except few selected libraries
which we do optimize).

```
export RUSTFLAGS="-C link-arg=-Tpal/pal_nrf/link.x -C linker-plugin-lto"
cargo bloat --target=thumbv7em-none-eabihf -- -n 0 --split-std --profile dev --crates

File  .text     Size Crate
1.3%  23.0% 213.5KiB pal_nrf
0.7%  12.6% 116.8KiB smoltcp
0.4%   7.8%  72.5KiB der
0.4%   7.8%  72.2KiB core
0.4%   6.8%  62.8KiB x509
0.4%   6.5%  60.6KiB fobnail
0.3%   4.7%  43.6KiB num_bigint_dig
0.2%   3.3%  30.1KiB coap_lite
0.2%   3.1%  28.3KiB alloc
0.2%   2.9%  26.5KiB trussed
0.1%   2.5%  23.5KiB [Unknown]
0.1%   1.6%  14.7KiB salty
0.1%   1.5%  13.8KiB aes
0.1%   1.3%  12.3KiB rsa
0.1%   1.3%  12.2KiB flexiber
0.1%   1.2%  11.1KiB const_oid
0.1%   0.9%   8.7KiB poly1305
0.0%   0.8%   7.8KiB x501
0.0%   0.7%   6.4KiB littlefs2_sys
0.0%   0.6%   6.0KiB sha2
0.0%   0.6%   5.4KiB cbor_smol
0.0%   0.5%   4.9KiB p256_cortex_m4_sys
0.0%   0.5%   4.9KiB compiler_builtins
0.0%   0.5%   4.8KiB usb_device
0.0%   0.5%   4.4KiB sha1
0.0%   0.5%   4.4KiB spki
0.0%   0.4%   4.1KiB rtt_target
0.0%   0.4%   3.6KiB usbd_ethernet
0.0%   0.4%   3.4KiB serde
0.0%   0.4%   3.4KiB log
0.0%   0.4%   3.3KiB linked_list_allocator
0.0%   0.3%   2.8KiB p256_cortex_m4
0.0%   0.3%   2.8KiB serde_bytes
0.0%   0.2%   2.2KiB chacha20
0.0%   0.2%   2.1KiB des
0.0%   0.2%   2.1KiB littlefs2
0.0%   0.2%   1.7KiB postcard
0.0%   0.2%   1.5KiB memchr
0.0%   0.2%   1.5KiB ed25519
0.0%   0.1%   1.3KiB zeroize
0.0%   0.1%   1.2KiB postcard_cobs
0.0%   0.1%   1.2KiB block_padding
0.0%   0.1%   1.1KiB fixed
0.0%   0.1%     966B half
0.0%   0.1%     888B serde_cbor
0.0%   0.1%     884B pkcs1
0.0%   0.1%     816B cortex_m
0.0%   0.1%     770B rand_chacha
0.0%   0.1%     768B nrf_hal_common
0.0%   0.1%     764B cipher
0.0%   0.1%     732B num_traits
0.0%   0.1%     598B rand_core
0.0%   0.1%     568B pkcs8
0.0%   0.1%     526B ppv_lite86
0.0%   0.1%     522B nrf_usbd
0.0%   0.1%     508B heapless
0.0%   0.1%     488B hash32
0.0%   0.1%     486B libm
0.0%   0.0%     440B flagset
0.0%   0.0%     374B byteorder
0.0%   0.0%     372B digest
0.0%   0.0%     370B subtle
0.0%   0.0%     334B cortex_m_rt
0.0%   0.0%     314B spin
0.0%   0.0%     310B interchange
0.0%   0.0%     294B generic_array
0.0%   0.0%     280B cstr_core
0.0%   0.0%     236B managed
0.0%   0.0%     208B crypto_bigint
0.0%   0.0%     208B smallvec
0.0%   0.0%     180B heapless_bytes
0.0%   0.0%     174B cfb_mode
0.0%   0.0%     136B rand
0.0%   0.0%     132B hmac
0.0%   0.0%      78B crypto_mac
0.0%   0.0%      74B az
0.0%   0.0%      72B block_buffer
0.0%   0.0%      56B nrf52840_pac
0.0%   0.0%      52B delog
0.0%   0.0%      44B crypto_common
0.0%   0.0%      32B aead
0.0%   0.0%      32B block_modes
0.0%   0.0%      32B inout
0.0%   0.0%      30B embedded_hal
0.0%   0.0%      10B cosey
0.0%   0.0%       2B bare_metal
5.7% 100.0% 927.1KiB .text section size, the file size is 15.8MiB

Note: numbers above are a result of guesswork. They are not 100% correct and never will be.
```

## Optimized binary size

Using `optimize-level = "z"` greatly reduces binary size, `cargo-bloat` shows
369 KiB, but binary actually takes 479K (excluding persistent storage size).

```
File  .text     Size Crate
0.5%  17.3%  63.9KiB pal_nrf
0.3%  10.8%  39.9KiB smoltcp
0.2%   8.4%  30.9KiB fobnail
0.2%   7.0%  25.9KiB core
0.2%   6.8%  25.3KiB [Unknown]
0.2%   5.8%  21.4KiB x509
0.2%   5.4%  19.8KiB der
0.2%   5.2%  19.4KiB num_bigint_dig
0.1%   4.4%  16.4KiB trussed
0.1%   3.5%  12.9KiB salty
0.1%   2.9%  10.7KiB aes
0.1%   2.4%   8.7KiB alloc
0.1%   2.2%   8.3KiB coap_lite
0.0%   1.5%   5.5KiB sha2
0.0%   1.3%   4.9KiB rsa
0.0%   1.2%   4.6KiB compiler_builtins
0.0%   1.1%   4.0KiB littlefs2_sys
0.0%   1.1%   4.0KiB sha1
0.0%   1.0%   3.8KiB p256_cortex_m4_sys
0.0%   0.8%   3.1KiB cbor_smol
0.0%   0.8%   3.0KiB const_oid
0.0%   0.8%   3.0KiB flexiber
0.0%   0.7%   2.4KiB poly1305
0.0%   0.6%   2.2KiB smallvec
0.0%   0.6%   2.2KiB usbd_ethernet
0.0%   0.6%   2.1KiB chacha20
0.0%   0.5%   1.9KiB x501
0.0%   0.5%   1.9KiB p256_cortex_m4
0.0%   0.5%   1.7KiB des
0.0%   0.4%   1.6KiB spki
0.0%   0.4%   1.6KiB usb_device
0.0%   0.4%   1.3KiB littlefs2
0.0%   0.3%   1.3KiB interchange
0.0%   0.3%   1.1KiB linked_list_allocator
0.0%   0.3%     964B byteorder
0.0%   0.2%     836B rtt_target
0.0%   0.2%     732B hmac
0.0%   0.2%     624B cipher
0.0%   0.2%     578B digest
0.0%   0.1%     456B serde
0.0%   0.1%     438B subtle
0.0%   0.1%     352B serde_bytes
0.0%   0.1%     352B log
0.0%   0.1%     300B postcard
0.0%   0.1%     300B block_padding
0.0%   0.1%     280B pkcs8
0.0%   0.1%     230B heapless
0.0%   0.1%     220B pkcs1
0.0%   0.0%     160B crypto_mac
0.0%   0.0%     144B embedded_hal
0.0%   0.0%     132B rand_core
0.0%   0.0%     120B generic_array
0.0%   0.0%     112B cortex_m
0.0%   0.0%     110B ppv_lite86
0.0%   0.0%     102B rand_chacha
0.0%   0.0%     100B nrf_hal_common
0.0%   0.0%      96B cortex_m_rt
0.0%   0.0%      94B block_buffer
0.0%   0.0%      82B zeroize
0.0%   0.0%      72B managed
0.0%   0.0%      72B nrf_usbd
0.0%   0.0%      70B cstr_core
0.0%   0.0%      54B crypto_bigint
0.0%   0.0%      54B half
0.0%   0.0%      38B delog
0.0%   0.0%      32B serde_cbor
0.0%   0.0%      32B block_modes
0.0%   0.0%      30B num_traits
0.0%   0.0%      26B heapless_bytes
0.0%   0.0%      16B inout
0.0%   0.0%      16B aead
0.0%   0.0%       6B ed25519
0.0%   0.0%       4B rand
2.9% 100.0% 369.9KiB .text section size, the file size is 12.3MiB

Note: numbers above are a result of guesswork. They are not 100% correct and never will be.
```

Analysis shows that most of the space is taken by `pal_nrf`.

## Deeper analysis

```shell
$ cargo bloat --target=thumbv7em-none-eabihf -- -n 15 --split-std --profile dev  --filter pal_nrf
File .text    Size   Crate Name
0.0%  1.1%  4.1KiB pal_nrf interchange::Responder<I>::take_request
0.0%  1.1%  4.0KiB pal_nrf trussed::service::ServiceResources<P>::reply_to
0.0%  0.5%  1.7KiB pal_nrf trussed::service::attest::try_attest
0.0%  0.3%  1.1KiB pal_nrf usb_device::device::UsbDevice<B>::poll
0.0%  0.2%    732B pal_nrf <chacha20::chacha::ChaCha<R,MC> as cipher::stream::StreamCipher>::try_apply_keystream
0.0%  0.2%    640B pal_nrf littlefs2::fs::Filesystem<Storage>::read_dir_and_then
0.0%  0.2%    620B pal_nrf littlefs2::fs::Filesystem<Storage>::read_dir_and_then
0.0%  0.2%    612B pal_nrf littlefs2::fs::Filesystem<Storage>::read_dir_and_then
0.0%  0.2%    584B pal_nrf trussed::service::ServiceResources<P>::rng
0.0%  0.2%    572B pal_nrf <nrf_usbd::usbd::Usbd<T> as usb_device::bus::UsbBus>::write
0.0%  0.1%    532B pal_nrf pal_nrf::trussed::store::Store::attach_else_format
0.0%  0.1%    516B pal_nrf trussed::mechanisms::chacha8poly1305::<impl trussed::service::Encrypt for trussed::mechanisms::Chacha8Poly1305>::encrypt
0.0%  0.1%    516B pal_nrf pal_nrf::trussed::init
0.0%  0.1%    508B pal_nrf <core::iter::adapters::map::Map<I,F> as core::iter::traits::iterator::Iterator>::try_fold
0.0%  0.1%    472B pal_nrf littlefs2::fs::Filesystem<Storage>::read_dir_and_then
0.4% 12.6% 46.5KiB         And 473 smaller methods. Use -n N to show more.
0.5% 17.2% 63.6KiB         filtered data size, the file size is 12.3MiB
```

Output shows that crate is `pal_nrf`, yet module path begins with `trussed`,
probably code inline. Rather there isn't much we can optimize here, bloat is
caused by Trussed and it's dependencies.

The following libraries are critical and cannot be replaced. Maybe some of them
could be optimized by tweaking theirs features.

```shell
File  .text     Size Crate       Comment
0.5%  17.3%  63.9KiB pal_nrf     Platform Abstraction Layer
0.3%  10.8%  39.9KiB smoltcp     Our network stack
0.2%   5.4%  19.8KiB der         Required for X.509 certificates support
0.2%   5.8%  21.4KiB x509        Required for X.509 certificates support
0.2%   7.0%  25.9KiB core        Rust core library (absolutely unreplaceable)
0.2%   8.4%  30.9KiB fobnail     Fobnail application itself
0.1%   4.4%  16.4KiB trussed     Trussed
0.0%   0.3%   1.3KiB interchange Used by Trussed for IPC
```

Following libraries should not be need, either we need them for software
cryptography (which we are going to replace with hardware) or are pulled by
Trussed but we don't use them.

Removing only these libraries would save us around **64 KiB**.

```shell
File  .text     Size Crate              Comment
0.2%   5.2%  19.4KiB num_bigint_dig     Required for RSA support
0.1%   3.5%  12.9KiB salty              Used by Trussed (Ed25519), we don't
                                        use it anymore because most (all?) TPMs
                                        don't implement it.
0.1%   2.9%  10.7KiB aes
0.0%   1.3%   4.9KiB rsa
0.0%   0.5%   4.4KiB sha1               Pulled by Trussed, we don't use it.
0.0%   1.5%   5.5KiB sha2               SHA128, SHA256
0.0%   1.0%   3.8KiB p256_cortex_m4_sys Not used.
0.0%   0.7%   2.4KiB poly1305
```

Another quite big thing is the `alloc` crate together with
`linked_list_allocator`. Going heapless could save us **9,8 KiB**, however
currently we use libraries that do require heap.

```
File  .text     Size Crate
0.1%   2.4%   8.7KiB alloc
0.0%   0.3%   1.1KiB linked_list_allocator
```

## Summary

In total we may halve binary size by enabling optimizations and save more than
**64 KiB** of space by debloating Trussed, getting rid of unnecessary
dependencies, moving from SW to HW crypto.
