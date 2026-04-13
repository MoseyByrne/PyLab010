# Using `bleak` to Detect and Log Nearby Low Energy (LE) Bluetooth Devices

## Objective

By the end of this lab, you will learn how to:
1. Use the `bleak` library to scan for Bluetooth devices.
2. Check for known devices and log their presence in a `shelve` database.
3. Understand how to work asynchronously in Python.

---

# Prerequisite Knowledge

1. **ASyncIO**: [Learn a bit About ASyncIO First](ASyncIO_Introduction.md)

# Setting Up Your Environment
1. **Install Bleak Library**
    * Use `pip` to install the `bleak` library OR install via PyCharm > Settings... > Project > Python Interpreter
    ```bash
    pip install bleak
    ```
2. **Import Necessary Modules**
    * Open your Python environment and start by importing the required libraries:
    ```python
    import asyncio
    import shelve
    from bleak import BleakScanner
    from time import strftime, gmtime, sleep
    ```

---

# Scanning for Bluetooth Devices
1. **Write a Function to Scan for Devices**
    * Let's create an asynchronous function to scan for Bluetooth devices:
     ```python
     async def scan_for_devices():
         print("Scanning for Bluetooth devices...")
         devices = await BleakScanner.discover()
         for device in devices:
             print(f"Device Name: {device.name}, Address: {device.address}")
         return devices
     ```

2. **Test the Scanning Function**
    * Run the function using `asyncio.run()` to ensure it works:
     ```python
     asyncio.run(scan_for_devices())
     ```
    * Check that nearby Bluetooth devices are displayed with their names and addresses.
    * If the name of your device doesn't show up here, **pair** your device with your computer and re-scan.
        * Record your device name and address for the next step (i.e. creating KNOWN_DEVICES).

---

# Checking for Known Devices
1. **Create a List of Known Device Addresses**
    * Get together with classmates and look at your mobile phone bluetooth names to construct a list of Bluetooth addresses to monitor:
     ```python
     KNOWN_DEVICES = {
         "JBL Tune 520BT-LE-1": "2E4C301A-FF8F-692F-E945-01D27DBCD839",
         "JBL Tune 520BT-LE-2": "63971F11-682D-CF9F-6927-B40D1461895F"
     }
     ```

2. **Write a Function to Check for Known Devices**
    * Modify the scanning function to log when a known device is detected:
     ```python
     async def check_for_known_devices():
         devices = await BleakScanner.discover()
         nearby_known_devices = {}

         for device in devices:
             if device.address in KNOWN_DEVICES.values():
                 device_name = [name for name, addr in KNOWN_DEVICES.items() if addr == device.address][0]
                 print(f"{device_name} is nearby!")
                 nearby_known_devices[device_name] = device.address

         return nearby_known_devices
     ```

---

# Logging with `shelve`
1. **Set Up a `shelve` Database**
    * Import `shelve` for a simple key-value storage database (See this week's chapter):
     ```python
     import shelve

      def log_devices(devices):
         with shelve.open( "device_log" ) as db:
             for name, address in devices.items():
                 key = name + '_' + address
                 formatted_time = strftime( "%Y-%m-%d %H:%M:%S", gmtime())
                 if key in db:
                     time_list = db[key]
                     time_list.append(formatted_time)
                     db[key] = time_list
                 else:
                     db[key] = [formatted_time]
         print( "Logged devices in the shelf database." )
     ```

2. **Combine Everything and Run the Code**
    * Create a main function to scan and log devices:
     ```python
     async def scan():
         nearby_devices = await check_for_known_devices()
         if nearby_devices:
             log_devices( nearby_devices )
         else:
             print( "No known devices nearby." )
     
     
     def main():
         # asyncio.run( scan_for_devices() )
         while True:
             print("Scanning...")
             asyncio.run( scan() )
             print("sleeping till next loop...")
             sleep(15)
     
     if __name__ == '__main__':
         main()
     ```

---

# Verifying the Log
1. **Check the Contents of the `shelve` Database**
     ```python
     def main():
         # asyncio.run( scan_for_devices() )
         while True:
             print("Scanning...")
             asyncio.run( scan() )
             print("sleeping till next loop...")
             sleep(15)
             # This is the verifying portion 
             print("Verifying log...")
             with shelve.open( "device_log" ) as db:
                 for name, address in db.items():
                     print( f"{name}: {address}" )
     
     if __name__ == '__main__':
         main()
     ```

2. **(OPTIONAL +10) Cleanup and Deleting Logs**
    * Implement this in a function that runs every hour using `asyncio` and `asyncio.sleep(3600)`
     ```python
     with shelve.open("device_log") as db:
         db.clear()
     print("Cleared the device log.")
     ```
