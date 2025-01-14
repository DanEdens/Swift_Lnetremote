Haven't really worked on this. 
Did the inital research but we went in another direction.
May revist later.

Moscapsule
==========

MQTT Client for iOS written in Swift.  
This framework is implemented as a wrapper of Mosquitto library and covers almost all mosquitto features.  
It uses Mosquitto version 1.4.8.

Mosquitto
---------
[Mosquitto](http://mosquitto.org/ "Mosquitto") is an open source message broker that implements the MQ Telemetry Transport protocol versions 3.1 and 3.1.1.
MQTT provides a lightweight method of carrying out messaging using a publish/subscribe model.
Mosquitto is written in C language.

Installation
------------

### CocoaPods
[CocoaPods](http://cocoapods.org) is a Cocoa project manager. It is a easy way for to install framework, so I recommend to using it.  
Create your project on Xcode and specify it in your podfile;
```ruby
target 'MyApp' do
  use_frameworks!

  project 'MyApp.xcodeproj'
  platform :ios, '9.0'

  pod 'Moscapsule', :git => 'https://github.com/flightonary/Moscapsule.git'
  pod 'OpenSSL-Universal'

  target 'MyAppTests' do
    inherit! :search_paths
  end

  target 'MyAppUITests' do
    inherit! :search_paths
  end
end
```

and then run;
```
$ pod install
```

Also you can specify old swift version in the podfile, such as the following;
```ruby
pod 'Moscapsule', :git => 'https://github.com/flightonary/Moscapsule.git', :branch => 'swift2'
```

In order to import the framework in tests, you should select configuration files.  
a) Select your project and `info`.  
b) Change configuration files from none to Pods.debug/release.  
![Configuration File](https://flightonary.github.io/img/inst_with_cocoapods.png)

### Manual Installation
If you don't want to use CocoaPods, you can install manually.

a) Check out Moscapsule.  
```
$ git clone https://github.com/flightonary/Moscapsule.git
```
b) The framework depends on [OpenSSL](https://github.com/krzyzanowskim/OpenSSL "OpenSSL"). Before building it, you must checkout the submodule.
```
$ git submodule update --init
```
c) Create a Xcode project and Workspace if you don't have these.  
d) Open workspace and drag & drop your project and Moscapsule.xcodeproj into Navigation.  
e) Drag & drop Moscapsule.xcodeproj under your project tree in Navitaion.  
f) Select your project and `Build Phases`.  
g) Add Moscapsule in `Target Dependencies` and `Link Binary With Libraries` using `+` button.  

![Moscapsule Manual Installation](https://flightonary.github.io/img/mosq_install.png)


Quick Start
-----
Here is a basic sample.
```swift
import Moscapsule

// set MQTT Client Configuration
let mqttConfig = MQTTConfig(clientId: "cid", host: "test.mosquitto.org", port: 1883, keepAlive: 60)
mqttConfig.onConnectCallback = { returnCode in
    NSLog("Return Code is \(returnCode.description)")
}
mqttConfig.onMessageCallback = { mqttMessage in
    NSLog("MQTT Message received: payload=\(mqttMessage.payloadString)")
}

// create new MQTT Connection
let mqttClient = MQTT.newConnection(mqttConfig)

// publish and subscribe
mqttClient.publishString("message", topic: "publish/topic", qos: 2, retain: false)
mqttClient.subscribe("subscribe/topic", qos: 2)

// disconnect
mqttClient.disconnect()
```

The framework supports TLS_PSK, Server and/or Client certification.  
Here is a sample for server certificate.
```swift
import Moscapsule

// Note that you must initialize framework only once after launch application
// in case that it uses SSL/TLS functions.
moscapsule_init()

let mqttConfig = MQTTConfig(clientId: "server_cert_test", host: "test.mosquitto.org", port: 8883, keepAlive: 60)

let bundlePath = NSBundle(forClass: self.dynamicType).bundlePath.stringByAppendingPathComponent("cert.bundle")
let certFile = bundlePath.stringByAppendingPathComponent("mosquitto.org.crt")

mqttConfig.mqttServerCert = MQTTServerCert(cafile: certFile, capath: nil)

let mqttClient = MQTT.newConnection(mqttConfig)
```

Reference
-------
#### Controlling Connections
Create new connection;
```swift
// create MQTT Client Configuration with mandatory prameters
let mqttConfig = MQTTConfig(clientId: "cid", host: "test.mosquitto.org", port: 1883, keepAlive: 60)
// create new MQTT Connection
let mqttClient = MQTT.newConnection(mqttConfig)
```
As soon as MQTTClient instance is created client attempt to connect to the host. It can be delayed by setting connectImmediately to false.
```swift
let mqttClient = MQTT.newConnection(mqttConfig, connectImmediately = false)
mqttClient.connectTo(host: "test.mosquitto.org", port: 1883, keepAlive: 60)
```

Diconnect the connection;  
```swift
mqttClient.disconnect()
```

Reconnect to the same host;  
```swift
mqttClient.reconnect()
```

#### Messaging

Publish a string to MQTT broker;  
```swift
let msg = "message"
mqttClient.publish(string: msg, topic: topic, qos: 2, retain: false)
```

Publish a data to MQTT broker;  
```swift
let data = Data(bytes: [0x00, 0x01, 0x00, 0x00])
mqttClient.publish(data, topic: topic, qos: 2, retain: false)
```

Subscribe a topic on MQTT broker;  
```swift
mqttClient.subscribe("/path/to/topic", qos: 2)
```
Messages on subscribed topic can be received in callback as mentioned below.

Unsubscribe a topic;  
```swift
mqttClient.unsubscribe("/path/to/topic")
```

#### Callbacks
Callback for result of attempting to connect;
```swift
mqttConfig.onConnectCallback = { returnCode in
  if returnCode == ReturnCode.success {
    // something to do in case of successful connection
  }
  else {
    // error handling for connection failure
  }
}
```

Callback which is called when a client disconnected;
```swift
mqttConfig.onDisconnectCallback = { reasonCode in
  if reasonCode == ReasonCode.disconnect_requested {
    // successful disconnection you requested
  } else if ... {
    // other cases such as unexpected disconnection.
  }
}
```

Callback which is called when a client published successfuly;
```swift
mqttConfig.onPublishCallback = { messageId in
  // successful publish
}
```

Callback which is called when a client received message successfuly;
```swift
mqttConfig.onMessageCallback = { mqttMessage in
  if mqttMessage.topic == "/path/to/subscribed/topic" {
    NSLog("MQTT Message received: payload=\(mqttMessage.payloadString)")
  } else if ... {
    // something to do in case of other topics
  }
}
```

Callback which is called when a client subscrived topic successfuly;
```swift
mqttConfig.onSubscribeCallback = { (messageId, grantedQos) in
  NSLog("subscribed (mid=\(messageId),grantedQos=\(grantedQos))")
}
```

Callback which is called when a client unsubscrived topic successfuly;
```swift
mqttConfig.onSubscribeCallback = { messageId in
  NSLog("unsubscribed (mid=\(messageId)")
}
```

#### Connection Options
T.B.D.


License
-------
Moscapsule: The MIT License (MIT)  
Mosquitto: EPL/EDL license

Author
------
tonary <<nekomelife@gmail.com>>
