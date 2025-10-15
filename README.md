# UEFI Firmware for BLE-based Configuration and Onboarding

## Building

In order to test BLE and FDO functionality using a [QEMU](https://www.qemu.org) virtual machine, the virtual machine firmware must be built with specific features enabled.

For cross-platform deterministic builds this project uses [Earthly](https://earthly.dev).  After installing Earthly, which requires a container toolchain like [Docker](https://www.docker.com) or [Podman](https://podman.io), run the following from the base of the repository:

```sh
earthly +ovmf
```

The build artifacts will be found under a local `build/ovmf/{UNIX_TIMESTAMP}/*` directory created during the pipeline execution.

## Testing

Testing assumes that a physical USB Bluetooth dongle is attached and mapped to a QEMU virtual machine.  This project has performed physical interoperability testing using a Raytac MDBT50Q-RX USB dongle with the the [Zephyr HCI USB](https://docs.zephyrproject.org/latest/samples/bluetooth/hci_usb/README.html#bluetooth_hci_usb) sample firmware.

1. On your test system, create a working directory and a virtual disk directory and copy in the build artifacts.  Assuming an example build timestamp directory of `1749769444`:

    ```sh
    mkdir -p vm/vdisk

    # Optionally add a startup.nsh EFI shell script
    cp path_to/startup.nsh vm/vdisk

    # Copy OVMF files
    cp build/ovmf/1749769444/*.fd vm

    # Copy additional EFI drivers or applications
    cp path_to_efis/*.efi vm/vdisk
    ```

2. Lookup the USB Bus ID and Port number of your USB attached Bluetooth controller.  The below example shows a connected USB dongle at **Bus 2 and Port 4** on an Ubuntu laptop:

    ```sh
    lsusb 
    > Bus 002 Device 022: ID 2fe3:000b NordicSemiconductor USB-DEV

    # Search for Device 022 to get the port:
    lsusb -t|grep 022
    >    |__ Port 004: Dev 022, If 0, Class=[unknown], Driver=btusb, 12M
    ```

3. Start the VM, updating the `hostbus=#,hostport=#` parameters with the above identified values:

    ```sh
    sudo qemu-system-x86_64 \
       -nodefaults -M accel=kvm -cpu host \
       -device virtio-rng-pci \
       -boot menu=on,splash-time=10000 \
       -machine q35 -smp 2 -m 2048M \
       -usb \
       -device qemu-xhci \
       -device usb-host,hostbus=2,hostport=4 \
       -vga std \
       -netdev user,id=u1 -device e1000,netdev=u1 \
       -drive if=pflash,format=raw,readonly=on,file=$(pwd)/vm/OVMF_CODE.fd \
       -drive if=pflash,format=raw,file=$(pwd)/vm/OVMF_VARS.fd \
       -drive file=fat:rw:$(pwd)/vm/vdisk,if=virtio
    ```

You can add the following options to your `qemu-system-x86_64` command for more diagnostics:

```sh
    -object filter-dump,id=f1,netdev=u1,file=dump.pcap \
    -debugcon file:debug.log -global isa-debugcon.iobase=0x402 \
    -gdb tcp::1234 \
```

> [!TIP]
> You can use the UEFI menu (press ESC during boot) to change the boot order and boot options so that the VM always and automatically starts the EFI Shell

### Troubleshooting

If you are getting errors, the following EFI Shell commands may help narrow down the issue:

1. Check that the USB HCI Bluetooth controller is available:

    ```sh
    dh -p usbio
    ```

1. Confirm the system has an IP address assigned

    ```sh
    ifconfig -l
    ```
