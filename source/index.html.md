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
TEST TEST TEST
Welcome to Flomio!! Get familiar with the Flomio products and explore their features in the [Flomio Shop](https://flomio.com/shop/)

Flomio builds hardware and software solutions in the proximity ID space. With a focus on mobile platforms, Flomio makes it easy to integrate our readers into Apps for iOS and Android.

You can use the Flomio SDK alongside any of Flomio's line of NFC, BLE, and UHF RFID readers.

You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Getting Started

Flomio helps you add a reader to your app so you can scan a tag to get a unique ID or read data. If you still need a helping hand after this, feel free to [ask the flomies](https://flomio.com/forums/forum/ask-the-flomies/).

Our native libraries for Android and iOS lets you collect NFC / UHF RFID without having to deal with the stress of managing the low level interactions between the reader and the tag.

## Download the Libraries

[Download the latest Flomio SDK](https://www.dropbox.com/s/v4yiasrq443ws5p/FlomioSDKv2.1_beta.zip?dl=1)
and unzip it. 

# iOS Integration

Our iOS libraries let you easily collect your customers/items tag information in your iOS app. Follow the instructions below to add the SDK to your project. 

## Configure Project Settings

<aside class="notice">
Steps 6 and 7 are for Swift developers only. The rest are for everybody! :)
</aside>

1. Drag+drop the "iOS" folder on project in Xcode navigator.
Make sure the "Copy items if needed" option is selected 
and click "Create groups" and "Finish" in dialog.

2. You must configure your application with the following settings.  
Click the Project Navigator (on the left), then click the
application name under TARGET. `In Targets -> YourAppTarget -> Build Settings 
-> Linking -> Other Linker Flags add ‘-lc++’ and ’-ObjC’`

3. `In Targets -> YourAppTarget -> Build Options 
-> Enable Bitcode set to ‘No’`

4. (optional) `In Target -> Build Settings
-> Apple LLVM 7.0-Preprocessing -> Preprocessor Macros add ‘DEBUGLOG’`

5. `In Targets -> YourAppTarget -> General
-> Link Binary with Libraries, add MediaPlayer.Framework`

6. For Swift Developers: Add a bridge header file: `File -> New > File... > Header File`
and name it ViewController-Bridging-Header.h

7. For Swift Developers: `In Targets -> YourAppTarget -> Build Options
-> Swift Compiler - General -> Objective-C Bridging Header`,
add `ViewController-Bridging-Header.h`

## Initialize the Flomio SDK

```objective_c
------------------------------------------
// in YourViewController.h, add this to viewDidLoad
#import "FmSessionManager.h"

@interface ViewController : UIViewController <FmSessionManagerDelegate> {  
  FmSessionManager *readerManager;
}   

@end

------------------------------------------
// in YourViewController.m, add this to viewDidLoad
- (void)viewDidLoad {
  readerManager = [FmSessionManager sharedManager];
  readerManager.selectedDeviceType = kFloBlePlus; // For FloBLE Plus
  //kFlojackMsr, kFlojackBzr for audiojack readers
  readerManager.delegate = self;
  readerManager.specificDeviceId = nil; 
  //@"RR330-000120" use device id from back of device to only connect to specific device
  // only for use when "Allow Multiconnect" = @0

  NSDictionary *configurationDictionary = @{
    @"Scan Sound" : @1,
    @"Scan Period" : @1000,
    @"Reader State" : [NSNumber numberWithInt:kReadUuid], //kReadData for NDEF
    @"Power Operation" : [NSNumber numberWithInt:kAutoPollingControl], //kBluetoothConnectionControl low power usage
    @"Transmit Power" : [NSNumber numberWithInt: kHighPower],
    @"Allow Multiconnect" : @0, //control whether multiple FloBLE devices can connect
    };

  [readerManager setConfiguration: configurationDictionary];
  [readerManager createReaders];
}

```

```swift
------------------------------------------
// in ViewController-Bridging-Header.h, add
#import "FmSessionManager.h"


------------------------------------------
// in ViewController.swift, add

class ViewController: UIViewController, FmSessionManagerDelegate {

  var device : FmDevice?
  let readerManager : FmSessionManager = FmSessionManager()
  
  override func viewDidLoad() {
      super.viewDidLoad()
      
      readerManager.selectedDeviceType = DeviceType.floBlePlus
      //flojackBzr, flojackMsr, flojackAny
      readerManager.delegate = self
      readerManager.specificDeviceId = nil;
      let configurationDictionary : [String : Any] =
          ["Scan Sound" : 1,
           "Scan Period" : 1000,
           "Reader State" : ReaderStateType.readUuid.rawValue, //readData for NDEF
           "Power Operation" : PowerOperation.autoPollingControl.rawValue, //bluetoothConnectionControl low power usage
           "Transmit Power" : TransmitPower.highPower.rawValue,
           "Allow Multiconnect" : false]
      
      readerManager.setConfiguration(configurationDictionary)
      readerManager.createReaders()
  }

```
You must first configure your settings and initialize the SDK. Add the code from your language of choice (on the right) to your app to create the readers. 

Here is a description for each configuration item, some items are not relevant for some readers.

Configuration item | Description
--------- | -------
selectedDeviceType | Choose your device type, you may only use one device type at a time.
specificDeviceId | Use the device id from back of device (or deviceId property) to only connect to a certain bluetooth reader. This is only for use when 'Allow Multiconnect' = @0.
Scan Sound | Hear notifications from Flojack MSR.
Scan Period | Period of polling for Audiojack readers.
Power Operation | Determine power operation for FloBle Plus. The affects how startReader and stopReader control your FloBle Plus, either control bluetooth for low power operation or nfc polling for standard use.
Transmit Power | Control the power of the NFC polling on the FloBle Plus.
Allow Multiconnect | Control whether multiple FloBle devices can connect simultaneously 

## Listen for Reader Events

```objective_c

- (void)active {
  NSLog(@"App Activated");
}

- (void)inactive {
  NSLog(@"App Inactive");
}

- (void)didFindTagWithUuid:(NSString *)Uuid fromDevice:(NSString *)deviceId withAtr:(NSString *)Atr withError:(NSError *)error{
    dispatch_async(dispatch_get_main_queue(), ^{
        //Use the main queue if the UI must be updated with the tag UUID or the deviceId
        NSLog(@"Found tag UUID: %@ from device:%@",Uuid,deviceId);
    });
}

- (void)didFindTagWithData:(NSDictionary *)payload fromDevice:(NSString *)deviceId withAtr:(NSString *)Atr withError:(NSError *)error{
    dispatch_async(dispatch_get_main_queue(), ^{
        //Use the main queue if the UI must be updated with the tag data or the deviceId
        if (payload[@"Raw Data"]){
            NSLog(@"Found raw data: %@ from device:%@",payload[@"Raw Data"] ,deviceId);
        } else if (payload[@"Ndef"]) {
            NSLog(@"Found Ndef Message: %@ from device:%@",payload[@"Ndef"] ,deviceId);
        }
    });
}

- (void)didReceiveReaderError:(NSError *)error {
    dispatch_async(dispatch_get_main_queue(), ^{ // Second dispatch message to log tag and restore screen
       NSLog(@"%@",error); //Reader error
    });
}

- (void)didUpdateConnectedDevices:(NSArray *)connectedDevices {
    //The list of connected devices was updated
}

- (void)didChangeCardStatus:(CardStatus)status fromDevice:(NSString *)deviceId {
     //The card status has entered or left the scan range of the reader
    // Cardstatus:
    // 0:kNotPresent
    // 1:kPresent
    // 2:kReadingData
}
```

```swift

func inactive() {
        print("App Inactive")
    }
    
func active() {
        print("App Activated")
    }


func didFindTag(withUuid Uuid: String!, fromDevice deviceId: String!, withAtr Atr: String!, withError error: Error!) {
        DispatchQueue.main.async {
            if let thisUuid = Uuid, let thisDeviceId = deviceId {
                print("Did find UUID: \(thisUuid) from Device: \(thisDeviceId)")
            } else {
                print(error)
            }
        }
    }
    
func didFindTag(withData payload: [AnyHashable : Any]!, fromDevice deviceId: String!, withAtr Atr: String!, withError error: Error!) {
    DispatchQueue.main.async {
        let thisDeviceId = deviceId
        if let thisPayload = payload["Raw Data"] {
            print("Did find payload: \(thisPayload) from Device: \(thisDeviceId)")
        } else if let ndef = payload["Ndef"] {
            print("Did find payload: \(ndef) from Device: \(thisDeviceId)")
        } 
    }
}

func didUpdateConnectedDevices(_ devices: [Any]!) {
    //The list of connected devices was updated
}

func didChange(_ status: CardStatus, fromDevice device: String!) {
    DispatchQueue.main.async {
        //The card status has entered or left the scan range of the reader
        // Cardstatus:
        // 0:CardStatus.notPresent
        // 1:CardStatus.present
        // 2:CardStatus.readingData
    }
}

func didReceiveReaderError(_ error: Error!) {
    DispatchQueue.main.async {
        print("Error: \(error)")
    }
}
```

Add the FlomioSessionManager delegates to receive scan events and reader status changes. First you need to indicate that you will conform to the FmSessionManagerDelegate and then add the delegate methods.

### Delegates
 
 Here is a quick summary of the notifications your app will receive from the Flomio SDK. 

 Delegate | Description
--------- | -------
didFindTagWithUuid(uuid) withAtr(atr) | Tag has been detected by the reader while in Reader State:Read UUID mode
didFindTagWithData withAtr(atr) | Tag has been detected by the reader while in Reader State:Read Data mode
to be continued...

# Android Integration

# Common Questions

Q | A
--------- | -------
 Can the SDK run in the background? | We don't expect Apps to ever be able operate FloJacks in the background. The FloJack requires a heartbeat handshake in order to prevent it from going into sleep mode and it's not possible to handle from background. The active/inactive methods in the ViewController are meant to wake/sleep the reader during foregounding/backgrounding. The FloBLE products are able to operate from a backgrounded app state. Also several of the same FloBLE products can connect to the iOS device at once (max 7 connected FloBLEs at a time).


# Get Help

Please visit [the forums](https://flomio.com/forums/forum/ask-the-flomies/) for help and to see previous questions.
