**Updated filters, so you need to use instruction below. 
Youtube video will be added**



# OpenESS (Anenji 4kw/7.2kw)

An app that exports electricity consumption and other metrics from Chinese solar inverters to Home Assistant using the WiFi adapter attached to the inverters. [SmartESS mobile app](https://play.google.com/store/apps/details?id=com.eybond.smartclient.ess).


Thanks [Alexey Denisov](https://github.com/alexeyden/openess)

## Home Assistant configuration

1. Install add-on Mosquitto broker

2. Add new user in HA settings (login and pass will be used for MQTT connection configuration)

3. Copy current user token (Profile>Security>Long-lived access tokens>Create token) will be used for HA connection

4. Install add-on NetDaemon V6 (.NET 10)
    add repository https://github.com/net-daemon/homeassistant-addon

5. Configure add-on NetDaemon V6 (.NET 10):
    * Create folder netdaemon4 under /homeassistant/
    * Add Application assembly name: NetDaemonApps.dll
    * Network Show disabled ports and add 8899 for 10000/tcp line

## Compilation and debug

1. Install [Visual Studio Community](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&channel=Stable&version=VS18&source=VSLandingPage&cid=2500&passive=false) with .Net and C++ package
2. Create project with NetDaemon: Default project template named "Inverter"
3. Copy all *.cs files from /src folder to project foldes apps/HasModel/Inverter
4. Remove template apps:
  \apps\AppModel
  \apps\Extensions
  \apps\HassModel\HelloWorld
  \apps\HassModel\LightOnMovement
5. Modify appsettings.json:
```json
  {
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Warning"
    },
    "ConsoleThemeType": "Ansi"
  },
  "HomeAssistant": {
    "Host": "192.168.1.30",
    "Port": 8123,
    "Ssl": false,
    "Token": ""
  },
  "Mqtt": {
    "Host": "192.168.1.30",
    "Port": "1883",
    "UserName": "mqtt",
    "Password": "mqtt",
    "DiscoveryPrefix": "homeassistant"
  },
  "Inverter": {
    "Host": "192.168.1.166",
    "Port": "58899"
  },
  "NetDaemon": {
    "LocalPort": "8899",
    "ApplicationConfigurationFolder": "./apps"
  },
  "CodeGeneration": {
    "Namespace": "HomeAssistantGenerated",
    "OutputFile": "HomeAssistantGenerated.cs",
    "UseAttributeBaseClasses": "false"
  }
}
```
6. In the NetDaemon configuration tab in the network configuration area, change port 10000 to 8899.
7. Modify Program.cs
```C#
   // Program.cs
   await Host.CreateDefaultBuilder(args)
        .UseNetDaemonAppSettings()
        .UseNetDaemonDefaultLogging()
        .UseNetDaemonRuntime()
        .UseNetDaemonTextToSpeech()
        // add for mqtt
        .UseNetDaemonMqttEntityManagement() 
        // #8
        .ConfigureServices((_, services) =>
            services
                .AddAppsFromAssembly(Assembly.GetExecutingAssembly())
                .AddNetDaemonStateManager()
                .AddNetDaemonScheduler()
                // Add next line if using code generator
                .AddHomeAssistantGenerated()
        )
        .Build()
        .RunAsync()
        .ConfigureAwait(false);
```

8. Run debug or publish to HA.
 

## P.S.

The Application, once launched, automatically registers the <b> OffGrid Inverter Anenji </b> device with all registers [201-234]( https://github.com/HotNoob/PythonProtocolGateway/blob/main/protocols/eg4/eg4_3000ehv_v1.holding_registry_map.csv ).

After successful integration to Home Assistant recomendation close access to internet for WiFi doungle. (Increases connection stability)

If someone writes a more detailed instruction or makes a video on YouTube, it would be welcomed.
 
 
