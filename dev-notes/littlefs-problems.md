# Problems with LittleFS

The main problem with LittleFS is its space inefficiency (see LittleFS issue
[#645](https://github.com/littlefs-project/littlefs/issues/645)).

We are using 64 KiB for LittleFS storage, flash memory on nRF52840 is 1 MiB, but
most of the space has been consumed by firmware (see
[firmware-size.md](firmware-size.md) for more information).

We have 64 KiB divided into 16 4 KiB blocks, block size is the erase block size
of the underlying device. LittleFS reserves 2 blocks for each directory, so
having 16 blocks we can have up to 8 directories.

Directory structure is partially enforced by Trussed - after formatting LFS,
Trussed creates directories `/trussed/dat/` for storing it's internal data. For
each new client, Trussed creates directories `/<client name>/dat/`. Currently we
have only client, so we get following filesystem structure.

```
/
| -- trussed
|     | -- dat
|
| -- fobnail_client
      | -- dat
```

This takes 8 blocks which already is half of available space. Fobnail creates
its own directories for storing metadata and for certificate storage.

```
/
| -- trussed
|     | -- dat
|
| -- fobnail_client
      | -- dat
            | -- meta
            | -- cert
                  | -- Internet Widgits Pty Ltd
                        | -- 004a0b62bd13f2444b774e9e0535ae49c8200967bf32ad976c9e89861f537505
```

Now we got 7 directories taking 14 blocks, one block is reserved for superblock.
This way we run out of space even though we stored only one file having ~900
bytes. We can't create more directories, we may still be able to store files in
respective directories if theirs blocks aren't full - file may be inlined and
stored in the same block that directory, so we have some space available in one
directory but not in another.

This problem can be alleviated by using flatter directory structure, possibly
Trussed could be modified to keep client data not in `dat` directory but in
parent.

Certificate storage uses 2 level structure - we create directory named after
organization name (from certificate subject) an put there certificates with
matching organization name. This is purely an optimization (for faster cert
chain verification) and directory structure could be flattened.

## Replacing LittleFS

The reason we are using LittleFS is that Trussed uses it, but we believe that
Trussed could be adapted to work with another filesystem.

There is an embedded filesystem library called
[TicKV](https://docs.tockos.org/tickv/index.html). Contrary to LittleFS it is
implemented in Rust (LittleFS is implemented in C). It is resistant to power
loss and has wear levelling.

We need to check whether it doesn't suffer from same or similar problems as
LittleFS. Also TickV is not a full filesystem, according to
[TicKV README](https://github.com/tock/tock/tree/master/libraries/tickv#how-tickv-works)
TicKV can store key/value pairs, not files, also it isn't stable and on-disk
format may change at any time.


