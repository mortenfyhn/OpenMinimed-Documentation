# Bluetooth communication

The insulin pumps and any connected devices use Bluetooth Low Energy (BLE) for communication. The pumps act as Central, while the sensors act as Peripheral. This is the only way multiple devices can be connected to a pump at the same time. It also means that any app (such as Medtronic's MiniMed Mobile app or our own scripts) must act as Peripheral when talking to the pump.

Establishing an initial connection roughly works like this: The pump is instructed (by the user) to scan for Peripherals that want to connect to it. The Peripheral sends out advertising packets that are picked up by the pump. These include some information that the pump can use to decide whether this is a suitable device or not. The pump then connects to a selected Peripheral. So it is always the pump that initiates the actual connection.

Both devices then exchange some keys (known as _pairing_ step) and store them for later use. Most importantly, this allows them to reestablish a lost connection without user interaction.

The devices utilize standardized and custom GATT services and characteristics. The interesting data (as payload in specific characteristics) is encrypted using a Medtronic protocol called SAKE (Secure? Authenticated? Key Exchange?).

Please check out our overview of [pump's](pump-services.md) and [app's](app-services.md) GATT services as well as the [Bluetooth SIG's official specifications](https://www.bluetooth.com/specifications/specs/) for more information.


## Advertising details

### Initial pairing

For the initial connection, the Peripheral must include the following data in its advertising packets:

Data                       | Type | Value
---------------------------|------|-------
16-bit Service Class UUIDs | 0x03 | 0xfe82
Device Name                | 0x09 | "Mobile xxxxxxx"

This specific _Class UUID_ is Medtronic's custom SAKE service.

The _Device Name_ must start with "Mobile " (note the whitespace) and can then be followed by 0 to 7 standard ASCII characters. This is the name the pump will display for confirmation. It serves no other purpose and can basically be random.


### Reconnects

If the pump and a paired Peripheral are disconnected (devices are too far apart, Bluetooth is temporarily disabled etc.), the pump tries to reconnect. It scans for a known Peripheral with the following data in its advertising packets:

Data                       | Type | Value
---------------------------|------|-------
16-bit Service Class UUIDs | 0x03 | 0xfe81

Note that this uses a _different_ UUID than the advertising for pairing. The pump ignores the device name, so there is no need to include it here.

Since the pump does not know if the Peripheral is gone for long, it does not make sense for the pump to spend lots of its battery power on scanning for the Peripheral in short intervals. If the Peripheral's advertising packets are sent only every second or so, chances are high that the pump will miss them if it only scans for them every couple of seconds, too.

> [!NOTE]
> We do not know if the pump actually behaves that way (it is not trivial to measure when and how often the pump is actually scanning), but it would make sense for a battery-powered device.

We can reliably get the pump to reconnect if we let the Peripheral send advertising packets every 150 ms (or faster). Advertising with much longer intervals would usually _not_ get the pump to reconnect.

> [!IMPORTANT]
> The pump seems to only reconnect to devices that are using a Resolvable Private Address (RPA). The initial pairing also works with public addresses though. So if you have trouble with reconnects, this is one of the things to check.

Confirmed on hardware (780G + a NimBLE peripheral): the pump reconnects **by bonded address**, not by advertised payload. It connects and re-runs SAKE whenever the peripheral is connectable on the address it bonded to — even when the peripheral advertises an unrelated payload with no `0xfe81` and no RPA. The `0xfe81` + RPA path is only how the pump finds a peripheral that advertises a privacy RPA: it resolves the RPA using the peripheral's IRK, which must be distributed during pairing (Android does this by default; NimBLE distributes the LTK only unless configured to include the identity keys). So a peripheral cannot avoid reconnects by changing its advert — only by refusing the connection. After reconnecting, the pump always re-runs the full SAKE handshake; there is no session resume.


## MAC addresses

The sensor MACs start with DC:16:A2 (Medtronic Diabetes, https://standards-oui.ieee.org/oui/oui.txt), while the pump use the private OUI 00:23:f7. The lower 3 bytes are the sensor's serial number, converted to hexadecimal. For example:

    CGM GT1122867N → 1122867 = 0x112233
    → Its MAC address should be DC:16:A2:11:22:33.

There seems to be no such relation between a pump's MAC address and its serial number. The addresses look rather random.


## Device names

Pumps use their serial numbers of the form `NGxxxxxxxH` and translate them into the device name `Pump xxxxxxxH`, keeping the 7-digit number the same.


## Connection parameters

The pump (as central) dictates the connection parameters and does not renegotiate them. Observed on a 780G with a peripheral that had completed SAKE and was polling CGM:

    connection interval   100 (125 ms)
    slave latency         0
    supervision timeout   300 (3000 ms)

The pump refuses peripheral-initiated parameter updates. An L2CAP Connection Parameter Update Request for slave latency 1 or 4 (keeping the pump's own interval and timeout) is rejected with HCI error `0x3B` _Unacceptable Connection Parameters_, even though it satisfies the spec's `supervision_timeout > (1 + latency) × interval × 2` constraint with margin. The pump rejects the mechanism, not a particular value.

This matters for a battery-powered peripheral: latency 0 at a 125 ms interval means the radio wakes ~8 times a second for as long as the link is up. The [NOS service](nos-service.md) "Observation Mode" write carries interval, latency and supervision timeout, and is presumably the sanctioned mechanism the official app uses.
