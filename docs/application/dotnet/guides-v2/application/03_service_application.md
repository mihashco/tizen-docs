# Service Applications 

Service applications are Tizen .NET applications with no graphical user interface that run in the background. They can be very useful in performing activities (such as getting sensor data in the background) that need to run periodically or continuously, but do not require any user intervention.

The service application type allows to create reusable and independent parts which is really important in bigger projects where we can easily split responsibilities of the application to different parts. As an example, consider the speedometer application designed for average speed measurement. Service part in this case is responsible for reading speed from the device GPS module. It also calculates average speed. The UI communicates with the service when it is visible and shows measured values. That approach allows to reuse module between different applications such as a cyclist application, automotive solutions, sport activity apps. Core module with business logic works in the background and do designed things until it will be closed by the system evens like `OnLowMemory` or `OnLowBattery` where UI app could be accidentally closed by the user.

The main Service Application API features include: 
-  Application States:

    A Tizen native service application has several different states which it transitions through during its life-cycle

- Event Callbacks:
  
    The service application can receive both basic system events and application state change events. You can register handlers for these events to react them.

- Application behavior attributes
  
    You can determine your service application behavior at boot time and after abnormal terminations by using specific attributes which you can set in the application manifest file. 

Service application can be explicitly launched by a UI application. They can also be launched conditionally.

The can check the running service applications in the task switcher; however no events occur if the user selects a service application from the task switcher. The main menu does not contain icons for service applications. Multiple service applications can be running simultaneously with other service and UI applications.

## Application States

The following figure and table describe the service application states.

**Figure: Service Application Lifecycle**
![Service Lifecycle](media/service_app_state.png)

**Table: Service application states**

| State        | Description                         |
|--------------|-------------------------------------|
| `READY`      | Application is launched.            |
| `CREATED`    | Application starts the main loop.   |
| `RUNNING`    | Application runs in the background. |
| `TERMINATED` | Application is terminated.          |

Because a service application has no UI, neither does it have a pause state. Since Tizen 2.4, the service application can go into the suspended state. Basically, the service application is running in the background by its nature; so the platform does not allow running the service application unless the application has a background category defined in its manifest file. However, when the UI application that is packaged with the service application is running on the foreground, the service application is also regarded as a foreground application and it can be run without a designated background category.

## Background Categories

Since Tizen 2.4, an application is not allowed to run in the background except when it is explicitly declared to do so. The following table lists the background categories that allow an application to run in the background.

<a name="allow_bg_table"></a>
**Table: Allowed background application policy**

| Background category            | Description                              | Related APIs                             | Manifest file \<background-category\> element value |
|--------------------------------|------------------------------------------|------------------------------------------|------------------------------------------|
| Media                          | Playing audio, recording, and outputting streaming video in the background | Multimedia API (in [mobile](../../api/mobile/latest/group__CAPI__MEDIA__FRAMEWORK.html) and [wearable](../../api/wearable/latest/group__CAPI__MEDIA__FRAMEWORK.html) applications) | `media`                                  |
| Download                       | Downloading data with the Tizen Download-manager API | Download API (in [mobile](../../api/mobile/latest/group__CAPI__WEB__DOWNLOAD__MODULE.html) applications) | `download`                               |
| Background network             | Processing general network operations in the background (such as sync-manager, IM, and VOIP) | Sync Manager API (in [mobile](../../api/mobile/latest/group__CAPI__SYNC__MANAGER__MODULE.html) applications), Socket, and Curl API (in [mobile](../../api/mobile/latest/group__OPENSRC__CURL__FRAMEWORK.html) and [wearable](../../api/wearable/latest/group__OPENSRC__CURL__FRAMEWORK.html) applications) | `background-network`                     |
| Location                       | Processing location data in the background | Location API (in [mobile](../../api/mobile/latest/group__CAPI__LOCATION__FRAMEWORK.html) and [wearable](../../api/wearable/latest/group__CAPI__LOCATION__FRAMEWORK.html) applications) | `location`                               |
| Sensor (context)               | Processing context data from the sensors, such as gesture | Sensor API (in [mobile](../../api/mobile/latest/group__CAPI__SYSTEM__SENSOR__MODULE.html) and [wearable](../../api/wearable/latest/group__CAPI__SYSTEM__SENSOR__MODULE.html) applications) | `sensor`                                 |
| IoT Communication/Connectivity | Communicating between external devices in the background (such as Wi-Fi and Bluetooth) | Wi-Fi (in [mobile](../../api/mobile/latest/group__CAPI__NETWORK__WIFI__PACKAGE.html) and [wearable](../../api/wearable/latest/group__CAPI__NETWORK__WIFI__PACKAGE.html) applications) and Bluetooth API (in [mobile](../../api/mobile/latest/group__CAPI__NETWORK__BLUETOOTH__MODULE.html) and [wearable](../../api/wearable/latest/group__CAPI__NETWORK__BLUETOOTH__MODULE.html) applications) | `iot-communication`                      |

  > **Note**
  >
  > Since Tizen 4.0, even if the background network category is declared, the running application stops if the network is not connected.

### Describing the Background Category

An application with a background running capability must declare the background category in its manifest file:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns="http://tizen.org/ns/packages" api-version="2.4" package="org.tizen.test" version="1.0.0">
   <ui-application appid="org.tizen.test" exec="text" type="capp" multiple="false" taskmanage="true" nodisplay="false">
      <icon>rest.png</icon>
      <label>rest</label>
      <!--For API version 2.4 and higher-->
      <background-category value="media"/>
      <background-category value="download"/>
      <background-category value="background-network"/>
   </ui-application>
   <service-application appid="org.tizen.test-service" exec="test-service" multiple="false" type="capp">
      <background-category value="background-network"/>
      <background-category value="location"/>
   </service-application>
</manifest>
```

> **Note**
>
> The `<background-category>` element is supported since the API version 2.4. An application with a `<background-category>` element declared can fail to be installed on devices with a Tizen version lower than 2.4. In this case, declare the background category as `<metadata key="http://tizen.org/metadata/background-category/<value>"/>`.
> ```
> <?xml version="1.0" encoding="utf-8"?>
> <manifest xmlns="http://tizen.org/ns/packages" api-version="2.3" package="org.tizen.test" version="1.0.0">
>    <ui-application appid="org.tizen.test" exec="text" type="capp" multiple="false" taskmanage="true" nodisplay="false">
>       <icon>rest.png</icon>
>       <label>rest</label>
>       <!--For API version lower than 2.4-->
>       <metadata key="http://tizen.org/metadata/background-category/media"/>
>       <metadata key="http://tizen.org/metadata/background-category/download"/>
>       <metadata key="http://tizen.org/metadata/background-category/background-network"/>
>    </ui-application>
>    <service-application appid="org.tizen.test-service" exec="test-service" multiple="false" type="capp">
>       <metadata key="http://tizen.org/metadata/background-category/background-network"/>
>       <metadata key="http://tizen.org/metadata/background-category/location"/>
>    </service-application>
> </manifest>
> ```
>
> The `<metadata key="http://tizen.org/metadata/bacgkround-category/<value>"/>` element has no effect on Tizen 2.3 devices, but on Tizen 2.4 and higher devices, it has the same effect as the `<background-category>` element.

## Code snippets 

Following code snippets shows the service application backbone generated from the Tizen templates.
The service template creates callback stubs to fill. Service application type is defined in the `Tizen.Applications` namespace, so it should be included at the top of the file. 

```csharp
using Tizen.Applications;
```

The `ServiceApplication` class has no UI, so `OnPause()` and `OnResume()` are not defined. Besides of that you can see here similar callbacks to other app types like `OnCreate()` and `OnTerminate()` called lifecycle evens and `OnLowBattery()`, `OnLowMemory()`, `OnLocaleChanged()` called system events because they are generated by the operating system.

`OnCreate()` method is used to take necessary actions before the main event loop starts. Place the initialization code here.

```csharp
namespace servicesample
{
    class App : ServiceApplication
    {
        protected override void OnCreate()
        {
            base.OnCreate();
        }
```

`OnAppConrolReceived()` callback is the most important callback here because it is responsible for data exchange between the service app and other applications.

```csharp
        protected override void OnAppControlReceived(AppControlReceivedEventArgs e)
        {
            base.OnAppControlReceived(e);
        }
```

`OnTerminate()` Used to take necessary actions when the application is terminating. Release all resources, especially any allocations and shared resources, so that other running applications can fully any shared resources.
The following table lists the system events.

```csharp
        protected override void OnTerminate()
        {
            base.OnTerminate();
        }
```

The following system events are connected with system state changes.

`OnLowMemory()`	Used to take necessary actions in low memory situations.
Save data in the main memory to a persistent memory or storage, to avoid data loss in case the Tizen platform Low Memory Killer kills your application to get more free memory. Release any cached data in the main memory to secure more free memory.

```csharp
        protected override void OnLowMemory(LowMemoryEventArgs e)
        {
            base.OnLowMemory(e);
        }
```

`OnLowBattery()` Used to take necessary actions in low battery situations.
Save data in the main memory to a persistent memory or storage, to avoid data loss in case the power goes off completely. Stop heavy CPU consumption or power consumption activities to save the remaining power.

```csharp
        protected override void OnLowBattery(LowBatteryEventArgs e)
        {
            base.OnLowBattery(e);
        }
```

`OnLocaleChanged()` and `OnRegionFormatChanged()` are invoked when region or system language was changed.

```csharp
        protected override void OnLocaleChanged(LocaleChangedEventArgs e)
        {
            base.OnLocaleChanged(e);
        }

        protected override void OnRegionFormatChanged(RegionFormatChangedEventArgs e)
        {
            base.OnRegionFormatChanged(e);
        }
```

```csharp
        static void Main(string[] args)
        {
            App app = new App();
            app.Run(args);
        }
    }
}
```

## Application Attributes
Describe your service application attributes in the manifest file. The attributes determine the application behavior. The following code example illustrates how you can define the attributes:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns="http://tizen.org/ns/packages" api-version="6" package="org.tizen.example.servicesample" version="1.0.0">
  <profile name="common" />
  <service-application appid="org.tizen.example.servicesample"
					exec="servicesample.dll"
					type="dotnet"
					multiple="false"
					taskmanage="false"
					nodisplay="true"
                    api-version="8">
    <label>servicesample</label>
    <icon>servicesample.png</icon>
  </service-application>
</manifest>
```

- `auto-restart`

  If set to `true`, the application restarts whenever it terminates abnormally. If the application is running, it is launched after installing or updating the package.

  > **Note**
  >
  > This attribute is not supported on Tizen wearable devices. Since Tizen 2.4, this attribute is not supported on all Tizen devices. Because of this, the `auto-restart` attribute used in a lower API version package than 2.4 is ignored on devices with the Tizen platform version 2.4 and higher.

- `on-boot`

  If set to `true`, the application launches on boot time, and after installing or updating the package. The application does not start if this attribute is removed after updating the package.

  > **Note**
  >
  > This attribute is not supported on Tizen wearable devices. Since Tizen 2.4, this attribute is not supported on all Tizen devices. Because of this, the `on-boot` attribute used in a lower API version package than 2.4 is ignored on devices with the Tizen platform version 2.4 and higher.

The following table defines the behaviors resulting from the attribute combinations:

**Table: Attribute combinations**

| `auto-restart` | `on-boot` | After normal termination   | On forced close            | On Reboot                           | After package installation | After package update       |
|----------------|-----------|----------------------------|----------------------------|-------------------------------------|----------------------------|----------------------------|
| `FALSE`        | `FALSE`   | Not launched automatically | Not launched automatically | Not launched after reboot           | Not launched               | Not launched automatically |
| `FALSE`        | `TRUE`    | Not launched automatically | Not launched automatically | Launched automatically after reboot | Launched                   | Launched automatically     |
| `TRUE`         | `FALSE`   | Launched automatically     | Launched automatically     | Not launched after reboot           | Not launched               | Launched automatically     |
| `TRUE`         | `TRUE`    | Launched automatically     | Launched automatically     | Launched automatically after reboot | Launched                   | Launched automatically     |

## Related Information
  * Dependencies
    -   Tizen 4.0 and Higher