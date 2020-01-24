# Reading and cloning a NFC Card using PN532 module

How to configure the module using USB UART adapter, install the required packages and examples on how to read and clone a NFC Card. Tested on Ubuntu 19.10 and Kali Linux 2019.4.

Requirements:

 - PN532 Module
 - USB UART Adapter (e.g. Unicorn TTL)
 - Wires
 - RFID Card

## Hardware

The following table shows how to properly connect the PN532 with the USB UART Adapter:

| PN532| USB UART Adapter  |
|------|-------------------|
| GND 	   | 		GND		 |
| VCC 	   | 		3V		 |
| TXD 	   | 		RXD		 |
| RXD 	   | 		TXD		 |

> Pay attention on the TXD and RXD configurations. The USB Adapter has to receive (RXD) what the PN352 is transmitting (TXD) and so on.

## Software

Install the required softwares:

    sudo apt-get install libnfc-bin libnfc-examples


Connect the USB Adapter on the computer and find the attached port:

    dmesg

    [119764.212841] hub 2-2:1.0: USB hub found
    [119764.213666] hub 2-2:1.0: 15 ports detected
    [119764.533554] usb 2-2.2: new full-speed USB device number 4 using uhci_hcd
    [119764.654743] usb 2-2.2: New USB device found, idVendor=0403, idProduct=6001, bcdDevice= 6.00
    [119764.654746] usb 2-2.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [119764.654748] usb 2-2.2: Product: FT232R USB UART
    [119764.654761] usb 2-2.2: Manufacturer: FTDI
    [119764.654762] usb 2-2.2: SerialNumber: AI02Z9LU
    [119764.756830] usbcore: registered new interface driver usbserial_generic
    [119764.756857] usbserial: USB Serial support registered for generic
    [119764.772234] usbcore: registered new interface driver ftdi_sio
    [119764.772338] usbserial: USB Serial support registered for FTDI USB Serial Device
    [119764.772528] ftdi_sio 2-2.2:1.0: FTDI USB Serial Device converter detected
    [119764.772624] usb 2-2.2: Detected FT232RL
    [119764.777646] usb 2-2.2: FTDI USB Serial Device converter now attached to ttyUSB0

The last line of the dmesg command shows that the USB is attached to **ttyUSB0**

Edit the NFC settings:

    vi /etc/nfc/libnfc.conf

The file should looks like this:

    # configuration using file or environment variable
    #allow_autoscan = true
    
    # Allow intrusive auto-detection (default: false)
    # Warning: intrusive auto-detection can seriously disturb other devices
    # This option is not recommended, user should prefer to add manually his device.
    #allow_intrusive_scan = false
    
    # Set log level (default: error)
    # Valid log levels are (in order of verbosity): 0 (none), 1 (error), 2 (info), 3 (debug)
    # Note: if you compiled with --enable-debug option, the default log level is "debug"
    log_level = 2
    
    # Manually set default device (no default)
    # To set a default device, you must set both name and connstring for your device
    # Note: if autoscan is enabled, default device will be the first device available in device list.
    device.name = "PN532 board via UART"
    device.connstring = "pn532_uart:/dev/ttyUSB0"

Testing the connection:

    nfc-list
     
    nfc-list uses libnfc 1.7.1
    NFC device: pn532_uart:/dev/ttyUSB0 opened

To read a tag, issue the following command:

    nfc-pool

And bring the tag closer to the PN532 module just for a moment. If everything is right than the following information will appear:

    nfc-poll uses libnfc 1.7.1
    NFC reader: pn532_uart:/dev/ttyUSB0 opened
    NFC device will poll during 30000 ms (20 pollings of 300 ms for 5 modulations)
    ISO/IEC 14443A (106 kbps) target:
        ATQA (SENS_RES): 00  04  
           UID (NFCID1): DE  AD  BE  EF  
          SAK (SEL_RES): 08  
    nfc_initiator_target_is_present: Target Released
    Waiting for card removing...done.

The `DEADBEEF` is the UID of the tag.

To clone a card you just need to copy the chosen UID (deadbeef  in this example), approximate the generic unlocked tag to the PN532 module and issue the following command:

    nfc-mfsetuid deadbeef

After this just check if everything went right using the `nfc-list` command.
