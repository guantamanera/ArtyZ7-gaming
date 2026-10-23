# ArtyZ7-gaming
Only tested on Arty Z7-20 Rev B board.
img with linux image and cores in release link See how to create image below

# Arty Z7-20 console preview

This package boots a standalone HDMI menu and provides separate Sega Master
System, NES and experimental SNES FPGA cores. The menu loads one core at a time.
Linux handles ROM loading and USB input; the consoles run in FPGA logic.

**Use only a Digilent Arty Z7-20 (XC7Z020, 512 MiB DDR). Do not use this image
on an Arty Z7-10, an Arty A7 or another board.** The development hardware is
Arty Z7-20 Rev B. No external SDRAM module is required.

This is a preview, not a general compatibility guarantee. The individual
core images have been used on the development Arty. The SNES original
diagnostic runs on hardware; commercial SNES game compatibility and
enhancement chips are not established. This particular redistributable SD
image has been checked offline, not boot-tested on a newly flashed card.

## Package contents

| Item | Use |
|---|---|
| `arty-z7-20-preview.img` | Complete SD image: partition table, boot files, Linux, menu, all three cores and original diagnostics |
| `SHA256SUMS` | Checksums for the extracted package |
| `cores/{menu,sms,nes,snes}/` | Separate `.bit` and FPGA-manager `console.bin` / `console.manifest` pairs |
| `diagnostics/` | Original project diagnostics; no commercial games are included |
| `boot-files/` | Individual boot artifacts for recovery/reference; normally use the complete image |
| `core-descriptors/` | Reference copies of the core catalog already installed inside Linux |
| `sources/`, `licenses/` | Project/upstream sources, component notices and rebuild information |
| `tools/` | The preview image builder and preview-specific Linux overlay |
| `BUILD-INFO.json`, `VALIDATION.txt` | Build identity and offline verification results |

Keep the source and license directories with the package when redistributing it.
No development-machine SSH credentials or personal ROM collection is included.

## 1. Create the SD card

You need a microSD card of **at least 2 GB**; 8 GB or larger is convenient for ROMs.
Back up the card first. **Writing the image erases its existing partitions and
all data.** Do not use this procedure to update a card containing wanted ROMs.

1. Extract the preview archive to a folder on your computer.
2. Verify the checksums. On Linux, run `sha256sum -c SHA256SUMS` from that folder.
   On Windows PowerShell, run
   `Get-FileHash .\arty-z7-20-preview.img -Algorithm SHA256`
   and compare it with the image's entry in `SHA256SUMS`.
3. Insert the microSD into a card reader.
4. Open [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and choose
   the custom-image option, or use [balenaEtcher](https://etcher.balena.io/).
   Select **`arty-z7-20-preview.img`**, then select the microSD by its capacity
   and device name. Do not select a computer drive.
5. Write and verify the image. In Imager, skip OS customization: this is not
   Raspberry Pi OS and the preview already defines its login.
6. Safely eject the card. If Windows offers to format another partition,
   **cancel**: that is the Linux ext4 data partition.

Do not just copy the `.img` into a formatted card. The imaging tool must write
the entire image. No Vivado, JTAG cable or separate FPGA programming step is
needed to boot the supplied card.

## 2. Expand storage to fill your card

The image starts with a **512 MiB FAT32 `ARTYBOOT` partition** and a
**256 MiB ext4 `ARTYDATA` partition**. Extra card capacity is initially
unallocated. Expansion is not automatic.

The recommended procedure is **GParted**, on a Linux computer or the
[GParted Live](https://gparted.org/livecd.php) environment:

1. Back up any wanted data. Shut down the Arty and remove power before
   removing its microSD.
2. Put the card in the computer's reader and open GParted.
3. Select the **microSD device** using its capacity/model. Double-check that
   it is not the computer's boot disk or the USB drive running GParted Live.
4. Unmount any mounted partitions on that card.
5. Select **partition 2, ext4, label `ARTYDATA`** and choose **Resize/Move**.
6. Leave its **starting position unchanged**. Drag only its right edge to
   the end of the device, or set free space following to zero.
7. Apply the operation. GParted expands both the partition and its ext4
   filesystem. Wait for completion before ejecting.

**Do not delete/recreate the partitions, format `ARTYDATA`, move its left
edge or enlarge `ARTYBOOT`.** Windows Disk Management cannot perform this
ext4 expansion. Changing only the partition boundary with another tool is
not sufficient: the filesystem also needs resizing.

After booting the Arty, check:

```sh
df -h /data
```

The data partition should now occupy almost all the remaining card capacity.
The FAT boot partition stays approximately 512 MiB. Displayed usable capacity
is lower than the card's marketing capacity because of units and filesystem
overhead.

## 3. Boot and connect

1. With the Arty powered off, set its boot-mode jumper to **SD**, following
   the board's silkscreen/manual.
2. Insert the card. Connect a display to **HDMI OUT**, a USB keyboard to the
   USB host connector, and Ethernet to a trusted DHCP-enabled network.
3. Power the board using a suitable supply. The FPGA menu and Linux load
   automatically.
4. Select a console, then its included diagnostic, to check the setup.

DHCP assigns an address; the development board's address is not hard-coded
for recipients. Find the device in your router's DHCP leases, or attach a
USB serial terminal to the board's programming/UART connector at
**115200 baud, 8 data bits, no parity, 1 stop bit, no flow control**.
Log in through serial and run:

```sh
ip -4 addr show eth0
```

The hostname is `arty-` followed by six card-derived hexadecimal characters.
Ethernet gets a locally administered, card-derived MAC address, so separate
cards do not all use the development board's MAC.

## 4. Login and change the default password

**Preview login: username `root`, password `1`.**

This intentionally simple default is insecure. Use the preview on a trusted
LAN only, never expose SSH to the Internet, and change the password immediately.
From your computer, replacing `BOARD_IP` with the actual address:

```sh
ssh root@BOARD_IP
passwd
```

Follow the prompts. The preview's `passwd` wrapper saves the password database
under `/data/configs/preview/`, so the change survives reboot. Run `passwd`
normally; do not bypass the wrapper with `busybox passwd`.

Each card generates its own SSH host key at first boot and retains it on
`ARTYDATA`. No development host key or authorized client key is shipped.
On the first connection, you can compare SSH's fingerprint with this command
on the USB serial console:

```sh
ssh-keygen -lf /data/configs/preview/ssh_host_ed25519_key.pub
```

Reflashing/resetting the card removes its saved password and host identity.
Do not distribute a copy of a card after you have used it: it may contain
your password hash, private host key and personal ROMs.

Other arbitrary changes to the Linux root filesystem do **not** persist:
Linux runs from an initramfs. `/boot` and `/data` are persistent; this preview
specifically persists password changes and SSH host keys on `/data`.

## 5. Put ROMs in the right folders

Only copy games you are entitled to use. The package includes original
diagnostics, not the developer's commercial games.

| Console | Folder on running Arty | Expected extension |
|---|---|---|
| SMS | `/data/roms/sms/` | `.sms` |
| NES | `/data/roms/nes/` | `.nes` |
| SNES | `/data/roms/snes/` | `.sfc` or `.smc`, without copier headers |

For example, from a computer with an SSH client:

```sh
scp "My Game.sfc" root@BOARD_IP:/data/roms/snes/
ssh root@BOARD_IP sync
```

SFTP clients also work. On a Linux computer with the card mounted, put ROMs
in `ARTYDATA/roms/snes/` (or the corresponding console folder). The label's
root is already `/data`; **do not create an extra `data` directory inside it**.
Safely unmount before moving the card back to the board.

Re-enter the console's ROM list to refresh it after copying new files.

## 6. Where the separate core files belong

All cores are already installed in the image. The extra copies are for
recovery, inspection and later replacement, not an additional setup requirement.

| Package directory | Destination on running Arty | Destination in mounted `ARTYDATA` |
|---|---|---|
| `cores/menu/` | `/data/cores/menu/` | `cores/menu/` |
| `cores/sms/` | `/data/cores/sms/` | `cores/sms/` |
| `cores/nes/` | `/data/cores/nes/` | `cores/nes/` |
| `cores/snes/` | `/data/cores/snes/` | `cores/snes/` |

The loader uses **`console.bin` and `console.manifest` together**.
The `.bit` files are standalone FPGA artifacts for development; they are not
the files the running Linux reloader loads. Keep the menu pair installed:
it is the mandatory rollback image.

For a replacement, use a matching pair supplied for this platform. Back up
the existing pair first. The simplest method is to shut down, use a Linux
card reader, and copy both files into the appropriate `ARTYDATA/cores/NAME/`
directory. Eject safely, boot, then validate:

```sh
arty-core check snes
```

For an SSH update, stop the menu **before** replacing files:

```sh
/etc/init.d/S99sms stop
```

Upload the replacement under temporary names first. Publish `console.bin`
and then `console.manifest`, validate with `arty-core check NAME`, run
`sync`, and restart with `/etc/init.d/S99sms start`. If validation fails,
restore the backed-up pair before restarting. Never overwrite a core image
while another client may load it.

`boot-files/menu.bit` belongs at the root of the FAT `ARTYBOOT` partition
(`/boot/menu.bit` while running). Keep it as the startup menu; do not rename
an emulator `.bit` to `menu.bit`.

`core-descriptors/*.core` are reference copies. The supplied Linux image
already has the descriptors and loaders in `/usr/share/arty/cores/` and
`/usr/bin/`. Do not put descriptors into the ROM folders. Adding an entirely
new core requires compatible Linux software/metadata as well as a bitstream.

## 7. Controls and preview limits

Menu: arrows navigate, Enter/Z selects, Esc/X goes back, **F12 shows/hides
the menu**. F12 does not pause a running game; controls are neutral and audio
muted while the menu is visible.

SMS: arrows, Z/X action buttons, Enter pause.

NES: arrows, Z=A, X=B, Enter=Start, Right Shift=Select.

SNES: arrows, Z=B, X=A, A=Y, S=X, Q=L, W=R, Enter=Start,
Right Shift=Select. Only player one is connected.

Gamepad mappings are supported in software, but physical gamepad compatibility
is less established than keyboard operation.

**Battery saves are not persistent yet in the console loaders.** The presence
of `/data/saves/` directories does not imply save support. Do not rely on a
game keeping progress across reloads.

SNES currently supports base NTSC LoROM/HiROM cartridges within the loader's
strict size/header rules, and supported 8 MiB extended mappings. Files must
be power-of-two sizes from 32 KiB through 8 MiB. Super FX, SA-1 and other
enhancement-chip games are rejected; for example, Doom is not supported.
A `.smc` extension does not make a copier-header ROM acceptable.

The menu displays loader/hardware failures rather than silently running an
unsupported ROM. Check files without changing the running core using
`smsctl check FILE`, `nesctl check FILE` or `snesctl check FILE`.

For logs use `/var/log/arty-menu.log` and `/run/arty-menu-command.log`.
Use a core's `status` command only while that core is selected.
Do not bypass `arty-core` and write to the FPGA manager directly.

Before removing power, run `sync` and `poweroff`; wait for shutdown.
Back up `/data` before reflashing or replacing a card.
