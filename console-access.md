# Connect to Switch Console 
Every network lab needs a way to console into network devices, it is just inevitable.  The Raspberry Pi can make a great terminal/console server for your network lab.  Some of the larger units have 4 USB ports, which would allow you to connect up to 4 different devices simultaneously.

There are many programs you can use for managing connections, but I've always found the basic `screen` application meets the needs I have in my network lab.  The Raspberry Pi doesn't have it installed by default, but you can easily fix that. 

1. Install `screen` application 

    ```bash
    sudo apt-get install screen
    ```

1. Connect the USB serial cable to your RPi and network device.  

1. Find the serial connection in the `/dev/` directory.  Looking in this directory will show you MANY devices, and it can be a bit tricky to recognize the name of the serial connection to your devices.  For this example, I'm connecting to a Cisco Catalyst 3650 switch (WS-C3650-24TS-S).  Sometimes searching in the manual for the equipment will tell you what to look for.  But I've found the "fastest" way to find the name of the device is to compare the contents of the `/dev` directory before I plug in the USB cable to after.  Then the one that is "new" is my device.  

    ```bash
    ls -l /dev/*ACM*
    ```

    > Many (but not all) of the Cisco USB console ports use the `ttyACM` base for the name.  This is a good one to remember and look for. 

1.  With the device identified, you can connect to device using screen.  

    ```bash
    screen /dev/ttyACM0 9600
    ```

    > If your device is configured for a different console speed, just change the 9600 in the command.

1. After you are done interacting with the device, disconnect from device cleanly by pressing "Cntl-a" followed by typeing `:quit`.  This will quit screen and return you to the Linux shell.

    ```bash
    # Press Ctrl-a
    :quit
    ```