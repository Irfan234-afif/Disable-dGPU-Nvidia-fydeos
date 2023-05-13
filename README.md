
# How to Disable Nvidia dGPU in FydeOS to clean

leaving the dGPU on is a huge annoyance, as the dGPU drains the battery. on the official FydeOS website, it has been explained how to disable the dGPU but that was not enough for me, because it only blacklis the kernel modul and OS still detects the presence of the dGPU.

this way i found when understanding how chromeos brunch works disabling dGPU. in brunch there is a patch that runs the command file ```.sh``` when booting and there I found the file ```99-dgpu.sh``` and I applied it to FydeOS manually
## Source Code

This code of file ```99-dgpu.sh```

```sh
# This patch adds a udev rule which helps certain dGPUs to stay suspended (improves battery life and prevents overheating)

ret=0

cat >/roota/lib/udev/rules.d/99-nvidia_pm.rules <<NOUVEAUPM
# Remove NVIDIA USB xHCI Host Controller devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{power/control}="auto", ATTR{remove}="1"
# Remove NVIDIA USB Type-C UCSI devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{power/control}="auto", ATTR{remove}="1"
# Remove NVIDIA Audio devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x040300", ATTR{power/control}="auto", ATTR{remove}="1"
# Remove NVIDIA VGA/3D controller devices
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x03[0-9]*", ATTR{power/control}="auto", ATTR{remove}="1"
NOUVEAUPM
if [ ! "$?" -eq 0 ]; then ret=$((ret + (2 ** 0))); fi
cat >/roota/etc/modprobe.d/blacklist-nouveau.conf <<NOUVEAUBLACKLIST
blacklist nouveau
options nouveau modeset=0
NOUVEAUBLACKLIST
if [ ! "$?" -eq 0 ]; then ret=$((ret + (2 ** 1))); fi

exit $ret
```
if you understand with this code, do it manually in your terminal.

nb: after ```cat >``` delete ```/roota```
## Installation

this is the manual method of the code file or you could say the translation of the ```.sh``` file

run this code in your terminal with ```Ctrl + Alt + T``` type ```shell``` then ```sudo su```

run this command
```sh
  $ cat >/lib/udev/rules.d/99-nvidia_pm.rules <<NOUVEAUPM
  > ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{power/control}="auto", ATTR{remove}="1"
  > ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{power/control}="auto", ATTR{remove}="1"
  > ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x040300", ATTR{power/control}="auto", ATTR{remove}="1"
  > ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x03[0-9]*", ATTR{power/control}="auto", ATTR{remove}="1"
  > NOUVEAUPM
```
The above code works to remove nvidia pci bus.

then run this code again

```sh  
  $ cat >/etc/modprobe.d/blacklist-nouveau.conf <<NOUVEAUBLACKLIST
  > blacklist nouveau
  > options nouveau modeset=0
  > NOUVEAUBLACKLIST
```
then reboot your devices.

Check whether the dGPU has been successfully disabled or not

open terminal like before, then type...
```
  lspci -k | grep -EA3 'VGA|3D|Display'
```

if what appears is intel graphics then disabling nvidia dGPU has been successful.

and 
```
  lsmod | grep nouveau
```
if the terminal doesn't show anything, then the disabling is complete
## Conclusion

For the question, email apipiirpan@gmail.com or guestbinggo@gmail.com

