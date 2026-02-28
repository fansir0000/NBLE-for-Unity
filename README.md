# NBLE-for-Unity Support

A Unity plugin that supports simultaneous connections to multiple BLE devices.
Only supports the Android platform

## Features

- ✅ Scan for BLE devices
- ✅ Connect to multiple devices simultaneously
- ✅ Discover services and characteristics
- ✅ Read/Write characteristics
- ✅ Subscribe/Unsubscribe to notifications
- ✅ Support Android 6.0+ (API 23+)
- ✅ Compatible with Android 12+ new Bluetooth permissions

## Quick Start

### Basic Usage

```csharp
using BLE;
using System.Text;

public class MyBLEController : MonoBehaviour
{
    private BLEManager ble;
    
    // Modify these UUIDs according to your device
    private string serviceUUID = "0000ffe0-0000-1000-8000-00805f9b34fb";
    private string writeCharUUID = "0000ffe1-0000-1000-8000-00805f9b34fb";
    private string notifyCharUUID = "0000ffe1-0000-1000-8000-00805f9b34fb";

    void Start()
    {
        ble = BLEManager.Instance;

        // Register events
        ble.Initialized += (success) => Debug.Log($"Initialized: {success}");
        ble.DeviceFound += (device) => Debug.Log($"Device found: {device}");
        ble.Connected += OnConnected;
        ble.ServicesDiscovered += OnServicesDiscovered;
        ble.CharacteristicChanged += OnDataReceived;
        ble.CharacteristicWritten += (addr, uuid) => Debug.Log($"Write successful");

        // Initialize
        ble.Initialize();
    }

    void OnConnected(string address)
    {
        Debug.Log($"Connected: {address}, total {ble.ConnectedDeviceCount} devices");
    }

    void OnServicesDiscovered(string address, List<string> services)
    {
        // Enable notifications to receive data
        ble.EnableNotification(address, serviceUUID, notifyCharUUID);
    }

    void OnDataReceived(string address, string uuid, byte[] data)
    {
        // Process received data
        string text = Encoding.UTF8.GetString(data);
        Debug.Log($"Received from {address}: {text}");
    }

    // Send string data
    public void SendText(string address, string text)
    {
        byte[] data = Encoding.UTF8.GetBytes(text);
        ble.WriteCharacteristic(address, serviceUUID, writeCharUUID, data);
    }

    // Send hexadecimal data
    public void SendHex(string address, byte[] data)
    {
        ble.WriteCharacteristic(address, serviceUUID, writeCharUUID, data);
    }

    // Send write without response (faster)
    public void SendFast(string address, byte[] data)
    {
        ble.WriteCharacteristicNoResponse(address, serviceUUID, writeCharUUID, data);
    }
}
```

### Multi-Device Connection Example

```csharp
// Connect to multiple devices
ble.Connect("AA:BB:CC:DD:EE:01");
ble.Connect("AA:BB:CC:DD:EE:02");
ble.Connect("AA:BB:CC:DD:EE:03");

// Check connection status
bool isConnected = ble.IsDeviceConnected("AA:BB:CC:DD:EE:01");
int count = ble.ConnectedDeviceCount;

// Send data to all devices
foreach (var conn in ble.ConnectedDevices)
{
    ble.WriteCharacteristic(conn.Key, serviceUUID, writeCharUUID, data);
}

// Disconnect a single device
ble.Disconnect("AA:BB:CC:DD:EE:01");

// Disconnect all devices
ble.DisconnectAll();
```

## Data Transmission Examples

### Sending Strings

```csharp
string text = "Hello BLE";
byte[] data = Encoding.UTF8.GetBytes(text);
ble.WriteCharacteristic(address, serviceUUID, writeCharUUID, data);
```

### Sending Hexadecimal Commands

```csharp
// Send command: AA 01 02 03 55
byte[] command = new byte[] { 0xAA, 0x01, 0x02, 0x03, 0x55 };
ble.WriteCharacteristic(address, serviceUUID, writeCharUUID, command);
```

### Receiving Data

```csharp
// 1. First enable notifications
ble.EnableNotification(address, serviceUUID, notifyCharUUID);

// 2. Listen to data change events
ble.CharacteristicChanged += (addr, uuid, data) =>
{
    // Parse as string
    string text = Encoding.UTF8.GetString(data);
    
    // Or parse as hexadecimal
    string hex = BitConverter.ToString(data);
    
    Debug.Log($"Data received: {text} (HEX: {hex})");
};
```

## Unity Settings

1. **Player Settings**:
   - Minimum API Level: 23 (Android 6.0)
   - Target API Level: 33+ (Recommended)

2. **Permissions**: Android 12+ requires runtime Bluetooth permission requests

## Important Notes

1. **UUID Format**: Use full 128-bit UUIDs, e.g., `0000ffe0-0000-1000-8000-00805f9b34fb`
2. **Multiple Devices**: All operations require specifying the device address
3. **Notifications**: Must call `EnableNotification` before receiving data
4. **Lifecycle**: Call `Cleanup()` in `OnDestroy` to release resources
5. **Main Thread**: All callbacks execute on the main thread, allowing direct UI updates