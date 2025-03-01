Deprecation warning
===================

This project is being deprecated in favour of the new integration (https://github.com/danielperna84/custom_homematic) based on hahomematic (https://github.com/danielperna84/hahomematic).  

If you are a user of the RaspberryMatic add-on, please follow these very detailed instructions to install the new integration as a custom component: https://github.com/jens-maus/RaspberryMatic/wiki/HomeAssistant-Integration  

Less detailed instructions also suitable for other installation types: https://github.com/danielperna84/custom_homematic/wiki/Installation  

Essentially the process is the same. When using a real CCU or RaspberryMatic Home Assistants auto-discovery may already detect your hub-device and you only have to provide credentials.

If you encounter problems with the new integration, please report them in the hahomematic repository.  

If you add a new device to your CCU and don't see the name you have set reflected in Home Assistant, please follow these instructions: https://github.com/danielperna84/hahomematic/wiki/Howto's

Other interesting information can be found here: https://github.com/danielperna84/hahomematic/tree/devel/docs

Please note, that this integration is -by the time of writing (May 2023)- still the default intergration for Homematic in Homeassistant.

Workaround for HmIPW-WRC2
=========================
The latest version of the original repository (0.1.77) lacks support for the HmIPW-WRC2 (https://de.elv.com/homematic-ip-wired-wandtaster-hmipw-wrc2-2-fach-154286) which has been patched in this repostiory.

If you can't or don't want to migrate to the new integration but need support for the device, you may host the patched devicetypes/misc.py-file on a local webserver or directly load from github and configure the "Run On Startup.d"-integration (https://community.home-assistant.io/t/run-on-startup-d/271008) to run a startup-job for your homeassistant docker container like e.g.: 

    >>> echo "This script is executed in the homeassistant container"; 
    >>> # Try to download the patched version of pyhomematic
    >>> wget -T 10 https://raw.githubusercontent.com/derco0n/pyhomematic/master/pyhomematic/devicetypes/misc.py -O /usr/local/lib/python3.10/site-packages/pyhomematic/devicetypes/misc.py
    >>> env;

pyhomematic
===========
.. image:: https://travis-ci.org/danielperna84/pyhomematic.svg?branch=master
    :target: https://travis-ci.org/danielperna84/pyhomematic

Python 3 Interface to interact with Homematic devices.

Important: This project will be deprecated in favor of https://github.com/danielperna84/hahomematic at some point in the future. A custom component to work with hahomematic can be found here: https://github.com/danielperna84/custom_homematic. Feel free to contribute!

This library provides easy (bi-directional) control of Homematic devices hooked up to a regular CCU or Homegear. The focus is to be able to receive events. If you are only interested in actively controlling devices, you can use the Python-built-in xmlrpc.client.ServerProxy (Python 3). See pyhomematic._server.ServerThread.connect on how to connect to a CCU / Homegear as a client.

Included is a XML-RPC server to receive events emitted by devices. Multiple callback functions can be set for devices to handle events. You can choose to bequeath callbacks from devices to their channels or not. Channels can not bequeath to their parent devices. You can also pass a callback funtion when creating the server, which then will (additionally) receive all events emitted by any paired device.

You may specify a devicefile (JSON) to store known devices. This might speed up startup a bit. If you don't, paired devices will always be propagated upon startup. If devices get paired while the server is running, they should be automatically detected and usable. To get notified about such events, it is possible to pass a systemcallback(source, *args)-function while creating the server.

Compatibility currently is only given for Python 3, since this library is primarily intended to add Homematic to https://home-assistant.io/, wich is written in Python 3.

As of now, usage is as follows (you could leave away the listening and remote addresses when everything is running on one machine):
    >>> def syscb(src, *args):
    >>>     print(src)
    >>>     for arg in args:
    >>>         print(arg)
    >>> def cb1(address, interface_id, key, value):
    >>>     print("CALLBACK WITH CHANNELS: %s, %s, %s, %s" % (address, interface_id, key, value))
    >>> def cb2(address, interface_id, key, value):
    >>>     print("CALLBACK WITHOUT CHANNELS: %s, %s, %s, %s" % (address, interface_id, key, value))
    >>> from pyhomematic import HMConnection
    >>> pyhomematic = HMConnection(local="192.168.1.12", localport=7080, remote="192.168.1.23", remoteport=2001, systemcallback=syscb) # Create server thread
    >>> pyhomematic.start() # Start server thread, connect to homegear, initialize to receive events
    >>> pyhomematic.devices['address_of_rollershutter_device'].move_down() # Move rollershutter down
    >>> pyhomematic.devices_all['address_of_doorcontact:1'].getValue("STATE") # True or False, depending on state
    >>> pyhomematic.devices['address_of_doorcontact'].setEventCallback(cb1) # Add first callback
    >>> pyhomematic.devices['address_of_doorcontact'].setEventCallback(cb2, bequeath=False) # Add second callback
    >>> pyhomematic.stop() # Shutdown to finish the server thread and quit

This example connects to the Homegear-server running on the same machine, closes the window shutter using the rollershutter device, queries the state of a door contact, adds callbacks for the door contact, then stops the server thread because a sample doesn't need to do more. The server has to be stopped because otherwise Python might hang.
An example.py can be found at https://github.com/danielperna84/pyhomematic

Theoretically all Homematic devices will be automatically detected and directly provide the getValue and setValue methods needed to perform any action.
Additionally, implemented devices provide convenience-properties and methods to perform certain tasks.

For more information visit the Wiki: https://github.com/danielperna84/pyhomematic/wiki

If pyhomematic doesn't seem to be what you need to interact with your CCU, then https://github.com/LarsMichelsen/pmatic might be (if Homegear support is not required).
