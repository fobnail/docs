# Use case: Disk encryption

## Description

Disk drive (either physical one or its image) contains personal or corporate
secrets. Access to that data must be restricted to limited set of users, even if
the drive itself may temporarily be made available to other parties, e.g. in
transit or left unattended in semi-public space like a hotel room.

### Threats

- Data theft - accessing secret information contained on disk
- Installment of unauthorized data - files on disk are overwritten or new files
are created

### Mitigations

- Key protection
    - Disk encryption key is off platform, it is on Fobnail Token
    - Remote attestation to Fobnail is need to obtain encryption key
- Platform integrity
    - Fobnail cryptographically bound to TPM
    - (FUTURE FEATURE) Ability to bind one Fobnail Token to multiple platforms

## Guide

This guide lists steps required to use Fobnail with LUKS2-formatted disk, but
other modes of encryption are also possible. It doesn't cover all features of
LUKS2, this is just an example of making it work with Fobnail. More advanced
usages can be found in [cryptsetup FAQ](https://gitlab.com/cryptsetup/cryptsetup/-/blob/master/FAQ.md),
and Fobnail doesn't limit any of those.

### Prerequisites

- [Provisioned](/token_provisioning) Fobnail Token
- At least one platform bound to that Token
    - (FUTURE FEATURE) May be more if same pair of disk/Token is to be used on
    multiple platforms
- Safe environment during initial preparation of disk

### Steps

1. (Optional - when using an image instead of physical disk)

    ```shell
    $ dd if=/dev/zero of=disk.img bs=1M count=128
    128+0 records in
    128+0 records out
    134217728 bytes (134 MB, 128 MiB) copied, 0,0718627 s, 1,9 GB/s
    $ sudo losetup -f --show disk.img
    /dev/loop5
    ```

    Last line is the name of newly created loop device. Use it in place of
    <your_disk> in following instructions.

2. **In secure environment** (passed attestation) create a keyfile that will be
used to decrypt master-key (see [LUKS2-docs](https://gitlab.com/cryptsetup/LUKS2-docs)
for details):

    ```shell
    $ dd bs=512 count=4 if=/dev/urandom of=/tmp/keyfile.bin
    4+0 records in
    4+0 records out
    2048 bytes (2,0 kB, 2,0 KiB) copied, 0,00431696 s, 474 kB/s
    ```

    You may use `/dev/random` if you're paranoid, but it may take much longer.
    Add `iflag=fullblock` in that case, otherwise key could be truncated.

    Make sure the environment is kept secure (e.g. don't leave the platform
    unattended) until further notice.

3. Initialize LUKS partition on the disk:

    ```shell
    $ sudo cryptsetup luksFormat --type luks2 /dev/<your_disk> keyfile.bin
    ```

    Despite `format` in its name, this command does not format the disk, but it
    still **destroys the data on it**. This will be reiterated by above command
    and you have to explicitly confirm it.

4. Map the LUKS2 container and create a file system on it:

    ```shell
    $ sudo cryptsetup luksOpen -d /tmp/keyfile.bin /dev/<your_disk> c1
    $ sudo mke2fs -j /dev/mapper/c1
    ```

    `c1` is the name of the mapping. It can be any other unused name. Another
    filesystems can also be used.

5. (Optional) Mount the partition and copy data to it:

    ```shell
    $ sudo mount /dev/mapper/c1 /mnt
    $ sudo cp top_secret_file.pdf /mnt/
    $ sudo umount /mnt
    ```

    It may be beneficial to call `sudo chown -R $USER:$USER /mnt` while the
    partition is mounted so accessing files would be possible as non-root user.

6. Close LUKS2 container:

    ```shell
    $ sudo cryptsetup close c1
    ```

7. Move key to Fobnail Token:

    ```shell
    $ sudo fobnail-attester --write-file /tmp/keyfile.bin:luks_key && \
      dd if=/dev/urandom of=/tmp/keyfile.bin bs=$(stat -c %s /tmp/keyfile.bin) count=1 && \
      rm /tmp/keyfile.bin
    ```

    `luks_key` is name under which file is saved on Fobnail Token. It can be
    arbitrary, as long as it doesn't contain any of the forbidden characters
    listed [here](/fobnail-api/#put-storagefsname).

    DO NOT overwrite or remove `keyfile.bin` unless it was successfully
    written to the Token or you won't be able to access the disk.

8. (Optional - when using an image instead of physical disk)

    ```shell
    $ sudo losetup -d /dev/<your_disk>
    ```

9. **At this point the keyfile should be present only in Fobnail Token. Platform
no longer has to be maintained in secure state.**

10. When access to the drive is required, plug in Token and disk and run:

    (Optional - when using image file) Repeat `sudo losetup -f --show disk.img`.
    Note that the device number may be different than previously.

    ```shell
    $ sudo fobnail-attester --read-file luks_key:- | \
      sudo cryptsetup luksOpen -d - /dev/<your_disk> c1
    $ sudo mount /dev/mapper/c1
    ```

    The same name as in step 7 must be used for reading, `luks_key` in this
    case. `-` in place of output filename tells to read to stdout, which is
    passed through a pipe to `cryptsetup`. This way the key isn't saved to disk.

11. Use drive as usual.

    Note that it will be accessible until the mapping is closed, regardless of
    Fobnail Token's presence or current platform state. Only guarantee is that
    at the time of requesting the keyfile attestation finished successfully.

12. When done, unmount and close the disk:

    ```shell
    $ sudo umount /mnt
    $ sudo cryptsetup close c1
    ```

13. (Optional - when using an image instead of physical disk)

    ```shell
    $ losetup -d /dev/<your_disk>
    ```
