---
title: API Reference

language_tabs:
  - objective_c
  - swift
  - java

toc_footers:

includes:

search: true
---

# Welcome

Welcome to Flomio!! Get familiar with the Flomio products and explore their features in the [Flomio Shop](https://flomio.com/shop/)

Flomio builds hardware and software solutions in the proximity ID space. With a focus on mobile platforms, Flomio makes it easy to integrate our readers into Apps for iOS and Android.

You can use the Flomio SDK alongside any of Flomio's line of NFC, BLE, and UHF RFID readers.

You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Getting Started

Flomio helps you build apps with NFC and RFID support so you can scan a tag to get a unique ID or read data. If after reading through this you still need a help, feel free to [ask the flomies](https://flomio.com/forums/forum/ask-the-flomies/).

Our Flomio SDK consists of native libraries for Android and iOS that let you collect NFC and RFID without having to deal with the stress of managing the low level interactions between the reader and tag. The Flomio SDK comes in 2 flavors, Basic and Pro. The Basic SDK checks for proper license on our cloud server so it requires connectivity to work. The Pro SDK works completely offline. More details in FAQ section below. 

To be sure your reader hardware is fully operational you should install the Flomio Test app via TestFlight. This will help resolve most issues you may encounter during your integration.

## Install Flomio Test app
You can now register to [download the Flomio Test app](https://flomio-mw.herokuapp.com/) via TestFlight. Once registered, you'll receive an email from the Apple TestFlight system directing you to install the TestFlight app and subsequently the Flomio Test app from within TestFlight. Once installed, the Flomio Test app will launch with a menu of our supported readers including:

+ FloJack BZR
+ FloJack MSR
+ FloJack Gen2
+ FloBLE EMV
+ FloBLE Plus

You will need to plug in the FloJack readers into the audio jack port of your iOS device or power on the FloBLE readers before selecting the appropriate reader from the menu. In order for the FloJack readers to operate well, make sure the volume of your iOS device is raised to the max. This is because the data is exchanged via audio modulation so the higher signal strength, the better data is captured by the reader. Also, the readers should be fully charged for best results. This is done by plugging in the readers to USB power and observing charging RED led indicator go off. At this point the device is fully charged. 

## Download the Framework

[Download the latest Flomio SDK](https://github.com/flomio/flomio-sdk-ios/releases) and unzip it. 

## Build the Example App

Example iOS projects can be seen on the [flomio-sdk-ios](https://github.com/flomio/flomio-sdk-ios) repo. 

## Cocoapods and Carthage 

To use the FlomioSDK with Cocoapods, follow the instructions from the [flomio-sdk-ios](https://github.com/flomio/flomio-sdk-ios) repo.

You can also manage your project's FlomioSDK dependency using Carthage. 

Follow [these instructions](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application) to get setup.

Add `github "https://github.com/flomio/flomio-sdk-ios"` to your Cartfile before running `carthage update`.
You will still need to configure your project settings by following the steps in the [Configure Project Settings](#configure-project-settings) section.

# iOS Integration

Our iOS libraries let you easily collect your customers/items tag information in your iOS app. Follow the instructions below to add the SDK to your project. 

## Configure Project Settings

1. Drag+drop the FlomioSDK.framework file into your Xcode Project.
Make sure the "Copy items if needed" option is selected 
and click "Create groups" and "Finish" in dialog.
It will be added to "Linked Frameworks and Libraries" section in project settings.

2. `In Targets -> YourAppTarget -> General -> Embedded Binaries, add the FlomioSDK`

3. You must configure your application with the following settings.  
Click the Project Navigator (on the left), then click the
application name under TARGET. `In Targets -> YourAppTarget -> Build Settings 
-> Linking -> Other Linker Flags add ‘-lc++’ and ’-ObjC’`

4. `In Targets -> YourAppTarget -> Build Options 
-> Enable Bitcode set to ‘No’`

5. `In Targets -> YourAppTarget -> Build Phases
-> Link Binary with Libraries, add MediaPlayer.Framework and libz.tbd`

6. (optional) `In Target -> Build Settings
-> Apple LLVM 9.0-Preprocessing -> Preprocessor Macros add ‘DEBUGLOG’`

## Initialize the Flomio SDK

See [FmConfiguration](#fmconfiguration) for more information on the configuration.

```objective_c
    ------------------------------------------
    // in YourViewController.h, add this to viewDidLoad
    #import <FlomioSDK/FlomioSDK.h>
    // For Pro Users
    // #import <FlomioSDKPro/FlomioSDKPro.h>
    
    @interface ViewController : UIViewController <FmSessionManagerDelegate> {  
        NSString *_deviceUid;
        FmSessionManager *flomioSDK;
    }   
    
    @end
    
    ------------------------------------------
    // in YourViewController.m, add this to viewDidLoad
    - (void)viewDidLoad {
        FmConfiguration *defaultConfiguration = [[FmConfiguration alloc] init];
        defaultConfiguration.deviceType = kFloBlePlus;
        defaultConfiguration.scanSound = @YES;
        defaultConfiguration.scanPeriod = @1000;
        defaultConfiguration.powerOperation = kAutoPollingControl;
        defaultConfiguration.transmitPower = kHighPower;
        defaultConfiguration.allowMultiConnect = @NO;
        defaultConfiguration.specificDeviceUid = nil;
        flomioSDK = [[FmSessionManager flomioMW] initWithConfiguration:defaultConfiguration];
        flomioSDK.delegate = self;
    }
       
```

```swift    

    import FlomioSDK 
    // For Pro Users
    // import FlomioSDKPro
    
    class ViewController: UIViewController, FmSessionManagerDelegate {
    
        var flomioSDK : FmSessionManager = FmSessionManager()
        var deviceUid = ""
        
        override func viewDidLoad() {
            super.viewDidLoad()
              
            let defaultConfiguration: FmConfiguration = FmConfiguration();
            defaultConfiguration.deviceType = DeviceType.kFloBlePlus;
            defaultConfiguration.transmitPower = TransmitPower.highPower;
            defaultConfiguration.scanSound = true;
            defaultConfiguration.scanPeriod = 1000;
            defaultConfiguration.powerOperation = PowerOperation.autoPollingControl;
            defaultConfiguration.allowMultiConnect = false;
            flomioSDK = FmSessionManager.init(configuration: defaultConfiguration);
            flomioSDK.delegate = self; 
        }

    }

```

## Listen for Reader Events

See [FmSessionManagerDelegate Methods](#fmsessionmanagerdelegate) for more information on the delegate methods.

```objective_c

    - (void)didFindTag:(FmTag *)tag fromDevice:(NSString *)deviceUid{
        [tag readNdef:^(FmNdefMessage *ndef) {
            if (ndef.ndefRecords) {
                for (FmNdefRecord *record in ndef.ndefRecords) {
                    if (record.url.absoluteString.length > 0){
                        // if there is a URL present in your tag, it will be here
                    }
                }
            }
        }];
    }
    
    - (void)didReceiveReaderError:(NSError *)error {
    
    }
    
    - (void)didChangeCardStatus:(CardStatus)status fromDevice:(NSString *)deviceUid{
    
    }
    
    - (void)didChangeStatus:(NSString *)deviceUid withConfiguration:(FmConfiguration *)configuration andBatteryLevel:(NSNumber *)batteryLevel andCommunicationStatus:(CommunicationStatus)communicationStatus withFirmwareRevision:(NSString *)firmwareRev withHardwareRevision:(NSString *)hardwareRev{
    }
    
    - (void)didGetLicenseInfo:(NSString *)deviceUid withStatus:(BOOL)isRegistered{
    
    }

```

```swift

    func didFind(_ tag: FmTag!, fromDevice deviceId: String!) {
        tag.readNdef { (ndefMessage) in
            guard let ndefRecords = ndefMessage?.ndefRecords else { return }
            for case let record as FmNdefRecord in ndefRecords {
               for case let record as FmNdefRecord in ndefRecords {
                   guard let url = record.url else { return }
                   // if there is a URL present in your tag, it will be here
               }
            }
        }
    }
    
    func didChangeStatus(_ deviceUid: String!, with configuration: FmConfiguration!, andBatteryLevel batteryLevel: NSNumber!, andCommunicationStatus communicationStatus: CommunicationStatus, withFirmwareRevision firmwareRev: String!, withHardwareRevision hardwareRev: String!) {
   
    }

    func didGetLicenseInfo(_ deviceUid: String!, withStatus isRegistered: Bool) {
    
    }
    
    func didChange(_ status: CardStatus, fromDevice device: String!) {
 
    }
    
    func didReceiveReaderError(_ error: Error!) {

    }
```

# Classes and Methods

## FmSessionManagerDelegate

Add FmSessionManagerDelegate to receive scan events and reader status changes.

Here is a quick summary of the notifications your app will receive from the Flomio SDK. 

 Delegate Method | Description
--------- | -------
didFindTag | Returns a [FmTag](#fmtag) object when a tag has been detected from the reader
didChangeStatus | Returns readers [CommunicationStatus](#communicationstatus) along with other reader info
didGetLicenseInfo | When using the basic SDK, this connected displays whether the reader is licensed
didChangeCardStatus | Returns a [CardStatus](#cardstatus) object when a tag has entered/left the readers range
didReceiveReaderError | Returns an error if there is a problem with the device

## FmSessionManager

 Method | Parameters | Description
--------- | ------- | -------
startReaders | | Enable paired or connected readers to begin polling for tags. This can mean different things depending on [PowerOperation](#poweroperation) setting.
stopReaders  | | Disable paired or connected readers from polling for tags. This can mean different things depending on [PowerOperation](#poweroperation) setting.
sleepReaders | | Put FloBLE Plus reader to sleep. This will also configure the reader to sleep after 60 seconds.
sendApdu toDevice success | APDU String, deviceId: String, completionBlock | Send an APDU using your connected device.
updateCeNdef withDeviceUid | ndef: [FmNdefMessage](#ndefmessage), deviceId: String | Configure your FloBLE Plus to emulate a tag with new NDEF data. Use when [FmConfiguration](#fmconfiguration)'s isCeMode is true.

## FmConfiguration

You must first configure your settings and initialize the SDK. 

Here is a description for each configuration item, some items are not relevant for some readers.

 FmConfiguration | Type | Description
--------- | -------  | -------
deviceType | [DeviceType](#devicetype) | Choose your device type, you may only use one device type at a time.
scanSound | Boolean | Hear notifications from Flojack MSR.
scanPeriod | Number |  Period of polling for Audiojack readers (ms).
powerOperation | [PowerOperation](#poweroperation) | Determine power operation for FloBle Plus. The affects how [startReader](#fmsessionmanager) and [stopReader](#fmsessionmanager) control your FloBle Plus, either control bluetooth for low power operation or nfc polling for standard use.
transmitPower | [TransmitPower](#transmitpower) | Control the power of the NFC polling on the FloBle Plus.
allowMulticonnect | Boolean |  Control whether multiple FloBLE devices can connect simultaneously.
specificDeviceUid | String | Use the device id from back of device (or deviceId property) to only connect to a certain bluetooth reader. This is only for use when 'Allow Multiconnect' = @0.
isCeMode | Boolean | Activates Card Emulation mode on FloBLE Plus

**Note:** Booleans listed are NSNumber initialized with Booleans. Use @YES/@No in Objective-C and true/false in Swift

## FmTag

This is returned when a tag is tapped from the didFindTag:tag fromDevice:deviceId delegate method on [FmSessionManagerDelegate](#fmsessionmanagerdelegate)

Properties

 FmTag | Type | Description
--------- | ------- | -------
atr | String | The ATR can be used to determine the type of tag.
uid | String | The Unique Identifier of the tag.

Methods

Method | Parameters | Description
--------- | ------- | -------
readNdef  | completionBlock | Returns the FmNdefMessage read from the tag in a completion block
writeNdef | FmNdefMessage, completionBlock | Pass a NDEF message to write, returns boolean to indicate whether tag was written successfully 

## FmNdefMessage
 `Object`

Represents an NDEF (NFC Data Exchange Format) data message that contains one or more [FmNdefRecords](#ndefrecord).

| Name | Type   | Description |
| --- | --- | --- |
| ndefRecords | Array of [FmNdefRecord](#ndefrecord)s | An array of [FmNdefRecord](#ndefrecord)s. |
| error | Error | Will explain the reason if there was a problem reading/parsing the data on the tag. |

## FmNdefRecord
 `Object`

Represents a NDEF (NFC Data Exchange Format) record as defined by the NDEF specification.

| Name | Type   | Description |
| --- | --- | --- |
| tnf | Number | The Type Name Format field of the payload. |
| type | Data | The type of the payload. |
| id | Data | The identifier of the payload |
| payload | Data  | The data of the payload |
| url | URL? | An optional convenience parameter which will return the payload as a URI for URI records. |


## DeviceType
 `enum`

 Name | Type       | Default | Description |
| --- | --- | --- | --- |
| kFlojackBzr | Number | `1` | FloJack BZR |
| kFlojackMsr | Number | `2` | FloJack MSR |
| kFloBleEmv | Number | `3` | FloBLE EMV |
| kFloBlePlus | Number | `4` | FloBLE Plus |
| uGrokit | Number | `5` | uGrokit |

## PowerOperation
 `enum`
 
 Parameter of [FmConfiguration](#fmconfiguration) for configuring the power of FloBLE Plus.

 Name | Type | Default | Description |
| --- | --- | --- | --- |
| AutoPollingControl | Number | `0` | startReaders and stopReaders turns on/off the reader from polling. |
| BluetoothConnectionControl | Number | `1` | startReaders and stopReaders turns on/off bluetooth, this can be used when exiting from/returning to your app. |


## CommunicationStatus
 `enum`
 
 This is returned when there is a status update from the didChangeStatus delegate method on [FmSessionManagerDelegate](#fmsessionmanagerdelegate)

 Name | Type | Default | Description |
| --- | --- | --- | --- |
| Scanning | Number | `0` | The device is connected and scanning for tags. |
| Connected | Number | `1` | The device is connected to bluetooth but not scanning for tags. |
| Disconnected | Number | `2` | The device is disconnected from bluetooth and not scanning for tags. |

## CardStatus
 `enum`
 
 This is returned when when a tag has entered/left the readers range from the didChangeCardStatus delegate method on [FmSessionManagerDelegate](#fmsessionmanagerdelegate)

 Name | Type | Default | Description |
| --- | --- | --- | --- |
| NotPresent | Number | `0` | A tag is not in range of the reader |
| Present | Number | `1` | A tag is in range of the reader. |

## TransmitPower
 `enum`
 
  Parameter of [FmConfiguration](#fmconfiguration) Used to determine the strength of the FloBLE Plus's transmit power.

 Name | Type | Default | Description |
| --- | --- | --- | --- |
| VeryLowPower | Number | `0` | Lowest power setting |
| LowPower | Number | `1` |  |
| MediumPower | Number | `2` |  |
| HighPower | Number | `3` | Highest power setting |

# Troubleshooting

Please visit [the forums](https://flomio.com/forums/forum/ask-the-flomies/) for help and to see previous questions.

## Unsupported Architecture
 
 When submitting to the App Store, if you get the Error `ERROR ITMS-90087: Unsupported Architecture. Your executable contains unsupported architecture '[x86_64, i386]'."`
 Add a Run Script step to your build steps, put it after your step to embed frameworks, set it to use `/bin/sh` and enter the following script:
 
 ```shell
 APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"
 
 # This script loops through the frameworks embedded in the application and
 # removes unused architectures.
 find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
 do
     FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
     FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
     echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"
 
     EXTRACTED_ARCHS=()
 
     for ARCH in $ARCHS
     do
         echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
         lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
         EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
     done
 
     echo "Merging extracted architectures: ${ARCHS}"
     lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
     rm "${EXTRACTED_ARCHS[@]}"
 
     echo "Replacing original executable with thinned version"
     rm "$FRAMEWORK_EXECUTABLE_PATH"
     mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"
 
 done
 ```
 
 credit: [Daniel Kennett](http://ikennd.ac/blog/2015/02/stripping-unwanted-architectures-from-dynamic-libraries-in-xcode/)

# Android Integration

# NFC Actions & IFTTT

The recently revamped [NFC Actions app](https://appsto.re/ie/AQ-bN.i) is now available in the App Store. 🎉🍾🥂

Use your [FloBle Plus](https://flomio.com/shop/readers/floble-plus-nfc-reader-for-mobile/) to write and read NFC tags (specifically Mifare Ultralight tags).

It also works with [IFTTT](https://ifttt.com/) so you can use NFC to automate your daily routine or prototype your NFC start-up without needing a front end app! 

For example, with IFTTT combined with NFC Actions, you can log employees time and attendance, you can monitor inventory or control smart devices with a tap! 

## Set up your IFTTT

### Create IFTTT Account 

First, you obviously need a [IFTTT account](https://ifttt.com/join). 

### Get Maker Webhook URL 

Then you'll need to go to get your Maker Webhook URL from the Maker Webhook settings. First connect [Maker Webhooks](https://ifttt.com/services/maker_webhooks/settings/connect) and then copy your Maker URL which should look like this: 

https://maker.ifttt.com/use/ciaOj1jhrldDVFHVn3XwwG-QTh18JHbQoiB3b-Cd02h

Side note (optional): Navigate to your Maker URL to see details about how to make HTTP POST requests to trigger events.

## Create your IFTTT Applet

You want to create your IFTTT Applet using Maker Webhooks to trigger your event... Sounds a lot more complicated than it is.

Go to [IFTTT Create](https://ifttt.com/create). Click "+this", search 'Maker Webhooks' and select it. Select 'Receive a web request' and name your event to, for example, 'tag_scanned'.

Side note (optional): For more precise control, you can use the each tag Uid to trigger different events by setting the Event name to the Uid. 

Now click '+that', we will use iOS Calendar as an example here but you can use what you want. Click 'create a calendar event'. Now configure your event to show the details of the tag scan. Type 'Now' into start time, click 'create' and 'finish'.   

Below are details of the values/'ingredients' sent with the trigger.

## IFTTT Ingredients 

 Ingredient | Description
--------- | -------
Value1 |  Tag Uid
Value2  | Tag Location in Google Maps
Value3 | Tag Payload (write to tag in NFC Actions to use this)

### Paste the URL in NFC Actions 

Now that you have your Maker URL and Maker Event Name, go to the [NFC Actions app](https://appsto.re/ie/AQ-bN.i) and navigate to settings and paste them into the IFTTT settings fields. 

# Frequently Asked Questions

Q | A
--------- | -------
 Can the SDK run in the background? | We don't expect Apps to ever be able operate FloJacks in the background. The FloJack requires a heartbeat handshake in order to prevent it from going into sleep mode and it's not possible to handle from background. The active/inactive methods in the ViewController are meant to wake/sleep the reader during foregrounding/backgrounding. The FloBLE products are able to operate from a backgrounded app state. Also several of the same FloBLE products can connect to the iOS device at once (max 7 connected FloBLEs at a time).
What's the difference between the Basic SDK and the Pro SDK? | The Flomio SDK is currently sold on the per device license. That means that if you build an app with the Flomio SDK it will only work with readers that have valid licenses. Every FloJack and FloBLE reader we sell includes the Basic SDK license. This license is checked against reader device ID every time your app runs by checking our online database. That means that you need have web connectivity for the Basic SDK to work. For customers that need more flexibility, the Pro SDK is able to operate completely offline. The license is checked inside the Pro SDK bundle itself so you're always guaranteed a good result. 
Can I read and write NDEF data to tags with my reader? | We recommend using the FloBLE Plus or FloBLE Noir to read and write data as the audio-jack data throughput is slow and unreliable. See [FmTag](#fmtag) for methods to read and write NDEF data.


#

Please visit [the forums](https://flomio.com/forums/forum/ask-the-flomies/) for help and to see previous questions.
