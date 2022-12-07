# Fix 25G port binding

Binding the Broadcom BCM57414 NetXtreme-E 10Gb/25Gb ports to the bnxt_en driver randomly fails with a time out error. Often, just one port on a dual port card will fail to bind (be claimed). Each reboot is different, so this script searches the dmesg output, looking for failures, then tries to bind the failed interface to the driver.

## Error and Success Messages
This shows port 1 on PCI address 0000:af:00.1 failing to bind.
```
[    2.497972] bnxt_en 0000:af:00.1 (unnamed net_device) (uninitialized): Error (timeout: 500) msg {0x0 0x0} len:176 v:0
[    2.498230] bnxt_en: probe of 0000:af:00.1 failed with error -1
```

A successful bind produces a rename log message, and sometimes this does still happen after a failure message.

A successful bind for port 0 on PCI address 0000:af:00.0, from the same boot, and same card.
```
[    2.530156] bnxt_en 0000:af:00.0 enp175s0f0: renamed from eth0
```

## Systemd
A one shot systemd startup script is run after the network startup has completed.

## Script
The script parses dmesg output, looking for the `failed with error` and `rename` messages and outputs the PCI addresses of the failed interfaces, so a loop can attempt to the bind the port to the bnxt_en driver.

A `rename` message takes precedence over a `failed with error` for the same PCI address, as sometimes the interface still binds after logging a timeout. This also permits the `fix_25G_port` script to be rerun, without producing an error message.

Tests indicate that trying to bind a port to the driver, when it is already bound, produces an error message, but does no harm to the running system.
