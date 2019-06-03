---
title: QR code tracking
description: Learn how to turn on QR code tracking for your Windows Mixed Reality headset and implement the feature in your apps.
author: dorreneb
ms.author: dobrown
ms.date: 05/15/2019
ms.topic: article
keywords: vr, lbe, location based entertainment, vr arcade, arcade, immersive, qr, qr code, hololens2
---

# QR Code Tracking

Specific Windows Mixed Reality devices can use QR codes to track in an environment. When enabled, the headset reports instances of QR codes found in an environment to your app.

>[!NOTE]
>The code snippets in this article currently demonstrate the use of C++/CX rather than C++17-compliant C++/WinRT as used in the [C++ holographic project template](creating-a-holographic-directx-project.md). The concepts are equivalent for a C++/WinRT project, though you need to translate the code.

## Device Compatibility for QR Code Tracking

### Hololens V1 Devices
Hololens V1 cannot natively track QR codes.

### Hololens V2 Devices
Hololens V2 devices can track QR codes by default. 

Apps that use this feature must request the webcam capability to read QR codes in a space.

### VR Devices

VR devices running Windows 10 version 1809 can track QR codes if the `HKLM\SOFTWARE\Microsoft\HoloLensSensors` registry key is enabled.

#### To turn on QR tracking:

1. Close the Mixed Reality Portal app on your PC.
2. Unplug the headset from your PC.
3. Open Command Prompt as administrator and run the following script: <br>
    `reg add "HKLM\SOFTWARE\Microsoft\HoloLensSensors" /v  EnableQRTrackerDefault /t REG_DWORD /d 1 /F`
4. Reconnect your headset to your PC.

#### To turn off QR tracking:

1. Close the Mixed Reality Portal app on your PC.
2. Unplug the headset from your PC.
3. Open Command Prompt as administrator and run the following script:<br>
    `reg add "HKLM\SOFTWARE\Microsoft\HoloLensSensors" /v  EnableQRTrackerDefault /t REG_DWORD /d 0 /F`

Completing these steps makes any discovered QR codes non-locatable.

>[!NOTE]
> Note that the version of QR tracking on Windows 10 version 1809 does not work on future versions of Windows, such as the version of Windows running on Hololens 2. If you migrate your app from VR to Hololens 2, you must update your QR tracking DLL.

## QRTracking API

The QRTracking plugin exposes the APIs for QR code tracking.

```cs
 // QRTracker plugin namespace
 namespace QRCodesTrackerPlugin
 {
    // Encapsulates information about a labeled QR code element.
    public class QRCode
    {        
        // Unique id that identifies this QR code for this session.
        public Guid Id { get; private set; }
           
        // Version of this QR code.
        public Int32 Version { get; private set; }
        
        // PhysicalSizeMeters of this QR code.
        public float PhysicalSizeMeters { get; private set; }
        
        // QR code Data.
        public string Code { get; private set; }
        
        // QR code DataStream this is the error corrected data stream
        public Byte[] CodeStream { get; private set; }
        
        // QR code last detected QPC ticks.
        public Int64 LastDetectedQPCTicks { get; private set; }
    };
    
    // The type of a QR Code added event.
    public class QRCodeAddedEventArgs
    {   
        // Gets the QR Code that was added
        public QRCode Code { get; private set; }
    };
    
    // The type of a QR Code removed event.
    public class QRCodeRemovedEventArgs
    {
        // Gets the QR Code that was removed.
        public QRCode Code { get; private set; }
    };
    
    // The type of a QR Code updated event.
    public class QRCodeUpdatedEventArgs
    {
        // Gets the QR Code that was updated.
        public QRCode Code { get; private set; }
    };
    
    // A callback for handling the QR Code added event.
    public delegate void QRCodeAddedHandler(QRCodeAddedEventArgs args);
    
    // A callback for handling the QR Code removed event.
    public delegate void QRCodeRemovedHandler(QRCodeRemovedEventArgs args);
    
    // A callback for handling the QR Code updated event.
    public delegate void QRCodeUpdatedHandler(QRCodeUpdatedEventArgs args);
    
    // Enumerates the possible results of a start of QRTracker.
    public enum QRTrackerStartResult
    {
        // The start has succeeded.
        Success = 0,
        
        //  The currently no device is connected.
        DeviceNotConnected = 1,
        
        // The QRTracking Feature is not supported by the current HMD driver
        // systems
        FeatureNotSupported = 2,
        
        // The access is denied. Administrator needs to enable QR tracker feature.
        AccessDenied = 3,
    };
    
    // QRTracker
    public class QRTracker
    {
        // Constructs a new QRTracker.
        public QRTracker(){}
        
        // Gets the QRTracker.
        public static QRTracker Get()
        {
            return new QRTracker();
        }
        
        // Start the QRTracker. Returns QRTrackerStartResult.
        public QRTrackerStartResult Start()
        {
            throw new NotImplementedException();
        }
        
        // Stops tracking QR codes.
        public void Stop() {}

        // Not Implemented
        Windows.Foundation.Collections.IVector<QRCode> GetList() { throw new NotImplementedException(); }
        
        // Event representing the addition of a QR Code.
        public event QRCodeAddedHandler Added = delegate { };
        
        // Event representing the removal of a QR Code.
        public event QRCodeRemovedHandler Removed = delegate { };
        
        // Event representing the update of a QR Code.
        public event QRCodeUpdatedHandler Updated = delegate { };
    };
}
```

## Using QR Code Tracking with the MRTK

## Implementing QR code tracking in Unity

You can use the QR Tracking API in Unity without taking a dependency on MRTK. To do so, you must:

1. Create a new folder in the assets folder of your unity project with the name *Plugins*.
2. Copy all the required files from this folder into the local "Plugins" folder you just created.

You can find an example of QR tracking in Unity [on Github here](
https://github.com/chgatla-microsoft/QRTracking/tree/master/AppPackages/SampleQRCodes_1.0.23.0_Master_Test).

## Implementing QR code tracking in DirectX

To use the QRTrackingPlugin in Visual Studio, you must add a reference of the QRTrackingPlugin to the .winmd. You can find the [required files for supported platforms here](https://github.com/Microsoft/MixedRealityToolkit-Unity/tree/htk_release/Assets/HoloToolkit-Preview/QRTracker/Plugins/WSA).

```cpp
// MyClass.h
public ref class MyClass
{
    private:
      QRCodesTrackerPlugin::QRTracker^ m_qrtracker;
      // Handlers
      void OnAddedQRCode(QRCodesTrackerPlugin::QRCodeAddedEventArgs ^args);
      void OnUpdatedQRCode(QRCodesTrackerPlugin::QRCodeUpdatedEventArgs ^args);
      void OnRemovedQRCode(QRCodesTrackerPlugin::QRCodeRemovedEventArgs ^args);
    ..
};
```

```cpp
// MyClass.cpp
MyClass::MyClass()
{
    // Create the tracker and register the callbacks
    m_qrtracker = ref new QRCodesTrackerPlugin::QRTracker();
    m_qrtracker->Added += ref new QRCodesTrackerPlugin::QRCodeAddedHandler(this, &OnAddedQRCode);
    m_qrtracker->Updated += ref new QRCodesTrackerPlugin::QRCodeUpdatedHandler(this, &OnUpdatedQRCode);
    m_qrtracker->Removed += ref new QRCodesTrackerPlugin::QRCodeRemovedHandler(this, &QOnRemovedQRCode);

    // Start the tracker
    if (m_qrtracker->Start() != QRCodesTrackerPlugin::QRTrackerStartResult::Success)
    {
      // Handle the failure
      // It can fail for multiple reasons and can be handled properly 
    }
}

void MyClass::OnAddedQRCode(QRCodesTrackerPlugin::QRCodeAddedEventArgs ^args)
{
    // use args->Code add to own list  
}

void MyClass::OnUpdatedQRCode(QRCodesTrackerPlugin::QRCodeUpdatedEventArgs ^args)
{
    // use args->Code update the existing one with the new one in own list 
}

void MyClass::OnRemovedQRCode(QRCodesTrackerPlugin::QRCodeRemovedEventArgs ^args)
{
    // use args->Code remove from own list.
}
```

## Getting a coordinate system

We define a right-hand coordinate system aligned with the QR code at the top left corner of the fast detection square in the top left as seen below. The Z-axis is pointing into the paper (not shown), but in Unity, the z-axis is out of the paper and left-handed.

A SpatialCoordinateSystem aligns as shown. This coordinate system can be obtained from the platform using the API `Windows::Perception::Spatial::Preview::SpatialGraphInteropPreview::CreateCoordinateSystemForNode`.

![QR code coordinate system](images/Qr-coordinatesystem.png) 

From QRCode^ Code object, the following code shows how to create a rectangle and put it in the QR coordinate system:

```cpp
// Creates a 2D rectangle in the x-y plane, with the specified properties.
std::vector<float3> SpatialStageManager::CreateRectangle(float width, float height)
{
    std::vector<float3> vertices(4);

    vertices[0] = { 0,  0, 0 };
    vertices[1] = { width,  0, 0 };
    vertices[2] = { width,  height, 0 };
    vertices[3] = { 0,  height, 0 };

    return vertices;
}
```

You can use the physical size to create the QR rectangle:

```cpp
std::vector<float3> qrVertices = CreateRectangle(Code->PhysicalSizeMeters, Code->PhysicalSizeMeters); 
```

The coordinate system can be used to draw the QR code or attach holograms to the location:

```cpp
Windows::Perception::Spatial::SpatialCoordinateSystem^ qrCoordinateSystem = Windows::Perception::Spatial::Preview::SpatialGraphInteropPreview::CreateCoordinateSystemForNode(Code->Id);
```

Altogether, your *QRCodesTrackerPlugin::QRCodeAddedHandler* may look something like this:

```cpp
void MyClass::OnAddedQRCode(QRCodesTrackerPlugin::QRCodeAddedEventArgs ^args)
{
    std::vector<float3> qrVertices = CreateRectangle(args->Code->PhysicalSizeMeters, args->Code->PhysicalSizeMeters);
    std::vector<unsigned short> qrCodeIndices = TriangulatePoints(qrVertices);
    XMFLOAT3 qrAreaColor = XMFLOAT3(DirectX::Colors::Aqua);

    Windows::Perception::Spatial::SpatialCoordinateSystem^ qrCoordinateSystem =  Windows::Perception::Spatial::Preview::SpatialGraphInteropPreview::CreateCoordinateSystemForNode(args->Code->Id);
    std::shared_ptr<SceneObject> m_qrShape =
        std::make_shared<SceneObject>(
            m_deviceResources,
            reinterpret_cast<std::vector<XMFLOAT3>&>(qrVertices),
            qrCodeIndices,
            qrAreaColor,
            qrCoordinateSystem);

    m_sceneController->AddSceneObject(m_qrShape);
}
```

## Considerations when creating tracking QR Codes

### Quiet zones around QR Codes

To be read correctly, QR codes require a margin around all sides of the code. This margin must not contain and printed content and should be four modules (a single black square in the code) wide. 

The [QR spec](https://www.qrcode.com/en/howto/code.html) contains more information about quiet zones.

### Lighting and backdrop
QR code detection quality is susceptible to varying illumination and backdrop. 

In a scene with particularly bright lighting, print a code that is black on a gray background. Otherwise, print a black QR code on a white background.

If the backdrop to the code is particularly dark, try a black on gray code if your detection rate is low. If the backdrop is relatively light, a regular code should work fine.

### Size of QR codes
WMR does not work with QR codes with sides smaller than 5 cm each.

For QR codes between 5 and 10 cm length sides, you must be fairly close to detect the code. It will also take longer to detect codes at this size. 

The exact time to detect codes depends not only on the size of the QR codes, but how far you are away from the code. Moving closer to the code will help offset issues with size.

### Distance and angular position from the QR code
The tracking cameras can only detect a certain level of detail. For really small codes - < 10cm along the sides - you must be fairly close. For a version 1 QR code varying from 10 to 25 cm wide, the minimum detection distance ranges from 0.15 meters to 0.5 meters. 

The detection distance for size increases linearly. 

QR detection works with a range of angles += 45deg. This is to ensure we have proper resolution to detect the code.

The following chart describes the minimum and maximum detection distances for QR tracking - the blue line shows the minimum distance we can detect the code at, whereas the orange line shows the maximum distance we can guarantee detection for. 

![QR code detection graph](images/qr-sizedistancegraph.png) 

### QR codes with logos
QR codes with logos have not been tested and are currently unsupported.

### Managing QR code data
WMR devices save QR codes in the boot session. Once you reboot the device or restart the driver, they are gone and will be re-detected as new objects next time. QR code history is saved at the system level in the driver session. It is recommended to configure your app to ignore QR codes older than a specific timestamp.

Currently, the API does not support clearing QR code history.
