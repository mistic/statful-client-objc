# Statful Client Objective-C

[![Build Status](https://travis-ci.org/statful/statful-client-objc.svg?branch=master)](https://travis-ci.org/statful/statful-client-objc)

Statful client for macOS and iOS written in Objective-C.
This client is intended to gather metrics and send them to the Statful service.

[//]: # (Please check out our [website](http://statful.com) or our extended [documentation](http://statful.com/docs) for a comprehensive look at all features available on Statful.)

## Table of Contents

* [Supported Platforms](#supported-platforms)
* [Installation](#installation)
* [Quick Start](#quick-start)
* [Examples](#examples)
* [Reference](#reference)
* [Still Need Help?](#still-need-help)
* [Authors](#authors)
* [License](#license)

## Supported Platforms

| Statful Client Version | Minimum iOS Target  | Minimum macOS Target  | Minimum watchOS Target  | Minimum tvOS Target  | Notes |
|:---|:---|:---|:---|:---|:---|
| 1.0.x | 8.0 | 10.8 | n/a | n/a | Xcode 7+ is required |

> **IMPORTANT:** Your project must support [64-bit with modern Cocoa runtime](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtVersionsPlatforms.html).

## Installation
Currently Statful Client Objective-C can only be installed with CocoaPods.

### CocoaPods

To install it using [CocoaPods](http://cocoapods.org), a dependency manager for Objective-C, ensure you have CocoaPods installed and configured on your project. 
If don't have it installed or configured in your project please check out [CocoaPods Documentation](https://guides.cocoapods.org).

#### Podfile

To integrate Statful Client into your Xcode project using CocoaPods, add this line to your `Podfile`:

```ruby
pod 'StatfulClientObjc', '~> 1.0.0'
```

And then run the following command:

```bash
$ pod install
```

#### Including Statful Client
At this point you only need to include Statful Client into your project to start using it. 

```objc
#import <StatfulClient/SFClient.h>
```

## Quick Start
After installing Statful Client you are ready to use it. The quickest way is to do the following:

```objc
#import <StatfulClient/SFClient.h>

// Creates a NSDictionary with configuration and pass it to the client
NSDictionary *clientConfig = @{@"transport":@(SFClientTransportAPI), @"token": @"YOUR_TOKEN_FOR_STATFUL_API"};
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];

// Starts the client to begin buffering and send metrics
// Any metric sent before the client start is ignored
[statfulClient start];

// Send a metric
[statfulClient counterWithName:@"testCounter" value:@0];

// Stop the client before exit your application to ensure you'll not loose metrics
// Every metric in the buffer is sent when client stop is called
[statfulClient stop];
```
> **IMPORTANT:** This configuration uses the default **host** and **port**. You can learn more about configuration in [Reference](#api-reference).

## Examples
You can find here some useful usage examples of the Statful Client. In the following examples is assumed you have already installed and included Statful Client in your project.

### UDP Configuration

Creates a simple UDP configuration for the client.

```objc
NSDictionary *clientConfig = @{ @"app": @"AccountService",
                                @"host": @"statful-relay.yourcompany.com",
                                @"tags": @{
                                    @"cluster": @"production"
                                },
                                @"transport":@(SFClientTransportUDP)
                             };
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];
```

### HTTP Configuration

Creates a simple HTTP API configuration for the client.

```objc
NSDictionary *clientConfig = @{ @"app": @"AccountService",
                                @"host": @"statful-relay.yourcompany.com",
                                @"tags": @{
                                    @"cluster": @"production"
                                },
                                @"token": @"YOUR_TOKEN_FOR_STATFUL_API",
                                @"transport":@(SFClientTransportAPI)
                             };
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];
```

### Logger Configuration

Creates a simple client configuration and change some logger's definitions.

```objc
NSDictionary *clientConfig = @{ @"host": @"statful-relay.yourcompany.com",
                                @"logger": [DDTTYLogger sharedInstance],
                                @"token": @"YOUR_TOKEN_FOR_STATFUL_API",
                                @"transport":@(SFClientTransportAPI)
                             };
                             
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];
                             
// Logger was instantiated with DDTTYLogger, a default implementation of Xcode Console Logger already provided by CocoaLumberjack. 
// Also is always instantiated with lower available logger level: SFLoggerLogLevelError.
// However you're able to change logger
statfulClient.logger.loggerLevel = SFLoggerLogLevelDebug;
                             
// You can also change the logger used by the client.
// For doing that you have to provide a logger that inherits from DDAbstractLogger <DDLogger>
// For example: 
statfulClient.logger.logger = [DDTTYLogger sharedInstance];

// Or 
DDFileLogger *fileLogger = [[DDFileLogger alloc] init];
fileLogger.rollingFrequency = 60 * 60 * 24;
fileLogger.logFileManager.maximumNumberOfLogFiles = 7;
statfulClient.logger.logger = fileLogger

```

### Defaults Configuration Per Method

Creates a configuration for the client with custom default options per method.

```objc
NSDictionary *clientConfig = @{ @"app": @"AccountService",
                                @"defaults": @{
                                    @"counter": @{
                                        @"agg": @[@"avg"],
                                        @"agg_freq": @180
                                    },
                                    @"gauge": @{
                                        @"agg": @[@"first"],
                                        @"agg_freq": @180
                                    },
                                    @"timer": @{
                                        @tags: @{
                                            @"cluster": @"qa"
                                        },
                                        @"agg": @[@"count"],
                                        @"agg_freq": @180
                                    }
                                },
                                @"host": @"statful-relay.yourcompany.com",
                                @"tags": @{
                                    @"cluster": @"production"
                                },
                                @"token": @"YOUR_TOKEN_FOR_STATFUL_API",
                                @"transport":@(SFClientTransportAPI)
                             };
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];
```

### Mixed Complete Configuration

Creates a configuration defining a value for every available option.

```objc
NSDictionary *clientConfig = @{ @"app": @"AccountService",
                                @"defaults": @{
                                    @"timer": @{
                                        @tags: @{
                                            @"cluster": @"qa"
                                        },
                                        @"agg": @[@"count"],
                                        @"agg_freq": @180
                                    }
                                },
                                @"dryrun": @YES,
                                @"flush_interval": @5000,
                                @"flush_size": @50,
                                @"host": @"statful-relay.yourcompany.com",
                                @"logger": [DDTTYLogger sharedInstance],
                                @"namespace": @"application",
                                @"port": @"123",
                                @"sample_rate": @95,
                                @"secure": @NO,
                                @"tags": @{
                                    @"cluster": @"production"
                                },
                                @"timeout": @300,
                                @"token": @"YOUR_TOKEN_FOR_STATFUL_API",
                                @"transport": @(SFClientTransportAPI)
                             };
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];
```

### Add Metrics

Creates a simple client configuration and use it to send some metrics.

```objc
NSDictionary *clientConfig = @{ @"host": @"statful-relay.yourcompany.com",
                                @"token": @"YOUR_TOKEN_FOR_STATFUL_API",
                                @"transport":@(SFClientTransportAPI)
                             };
SFClient *statfulClient = [SFClient clientWithConfig:clientConfig];

// Ignored metric
[statfulClient gaugeWithName:@"testTimer" value:@0];

[statfulClient start];

// Send three different metrics (gauge, timer and a counter)
// Attention: calling method gaugeWithName:value is equal to 
// calling gaugeWithName:value:options with null options
[statfulClient gaugeWithName:@"testTimer" value:@0];
[statfulClient timerWithName:@"testTimer" value:@0 options:nil];
[statfulClient timerWithName:@"testTimer" value:@0 options:@{@"agg": @[@"first"],
                                                             @"agg_freq": @60
                                                            }];
                                                            
// Metric to be sent with more custom options like timestamp instead of current timestamp
[statfulClient timerWithName:@"testTimer" value:@0 options:@{@"tags": @{
                                                                @"cluster": @"sandbox"
                                                             },
                                                             @"agg": @[@"last"],
                                                             @"agg_freq": @60
                                                             @"namespace": @"sandbox"
                                                             @"timestamp": @"1469714440"
                                                            }];
                                                            
[statfulClient stop];

// Ignored metric
[statfulClient gaugeWithName:@"testTimer" value:@0];

```

## Reference 

Our Statful Client API it's very simple. You can browse the full reference below.

### Client

The Client used to send metrics for the system.

#### Class 
`SFClient`

#### Enumerated Types

##### SFClientTransport

| Type | Description |
|:---|:---|
| `SFClientTransportAPI` | An enumerated type that defines API transport. It makes the client send the metrics through a HTTP API. |
| `SFClientTransportUDP` | An enumerated type that defines UDP transport. It makes the client send the metrics through an UDP socket. |

#### Methods

```objc
+ (instancetype)clientWithConfig:(NSDictionary*)config
```

This is a class method that receives a `NSDictionary *` with configuration  and returns a new `SFClient`.
The custom options that can be set on config param are detailed below.

| Option | Description | Type | Default | Required |
|:---|:---|:---|:---|:---|
| _app_ | Defines the application global option. If specified sets a global tag `app=settedValue`. | `NSString*` | **none** | **NO** |
| _defaults_ | Defines the global default options per method. | `NSDictionary*` | `@{}` | **NO** |
| _dryrun_ | Defines if metrics should be outputed to the logger instead of being sent. | `NSNumber*` | `@NO` | **NO** |
| _flush_interval_ | Defines the periodicity of buffer flush in **miliseconds**. | `NSNumber*` | `@10000` | **NO** |
| _flush_size_ | Defines the number of metrics that the buffer should have to make a flush. | `NSNumber*` | `@10` | **NO** |
| _host_ | Defines the host for which metrics should be sent. | `NSString*` | `@"127.0.0.1"` | **NO** |
| _logger_ | Defines the internal logger instance according to `DDLogger` protocol from [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) used by SFLogger to output the messages. It can be one already defined by CocoaLumberjack like `DTTYLogger`, `DDASLLogger`, `DDFileLogger` or any other custom logger object that complies with `DDAbstractLogger <DDLogger>` from CocoaLumberjack. | `DDAbstractLogger <DDLogger> *` | **none** | **NO** |
| _namespace_ | Defines the global namespace. | `NSString*` | `@"application"` | **NO** |
| _port_ | Defines the host por for which metrics should be sent. | `NSString*` | `@"2013"` | **NO** |
| _sample_rate_ | Defines the rate sampling. **Should be a number between [1, 100]**. | `NSNumber*` | `@100` | **NO** |
| _secure_ | Enable or disable https protocol. | `NSNumber*` | `@"true"` | **NO** |
| _tags_ | Defines the global tags. | `NSDictionary*` | `@{}` | **NO** |
| _timeout_ | Defines the timeout for the transport layers in **miliseconds**. | `NSNumber*` | `@2000` | **NO** |
| _token_ | Defines the token to be used  `SFClientTransportAPI`.  | `NSString*` | **none** | **NO** |
| _transport_ | Defines the transport layer to be used to send metrics.<br><br> **Valid Transports:** `SFClientTransportAPI, SFClientTransportUDP` | `NSNumber*` | **none** | **YES** |

```objc
- (BOOL)start
```

This method tries to start the client and returns a boolean. If it succeeds it becomes possible send metrics. 

```objc
- (BOOL)stop
```

This method tries to stop the client and also send all the metrics still in the buffer returning a boolean. If it succeeds it becomes impossible send metrics. 

```objc
- (void)counterWithName:(NSString*)name value:(NSNumber*)value
- (void)gaugeWithName:(NSString*)name value:(NSNumber*)value
- (void)timerWithName:(NSString*)name value:(NSNumber*)value
```

These method receives a string name and a number value and sends a simple counter/gauge/timer metric (with default method options).
Read the reference of methods with options params to know the defaults.

```objc
- (void)counterWithName:(NSString*)name value:(NSNumber*)value options:(NSDictionary*)options
- (void)gaugeWithName:(NSString*)name value:(NSNumber*)value options:(NSDictionary*)options
- (void)timerWithName:(NSString*)name value:(NSNumber*)value options:(NSDictionary*)options
```

These method receives a string name, a number value, a dictionary with options and sends a counter/gauge/timer metric with custom options.
The custom options that can be set on options param are detailed below.

| Option | Description | Type | Default for Counter | Default for Gauge | Default for Timer |
|:---|:---|:---|:---|:---|:---|
| _agg_ | Defines the aggregations to send with metric. It merges with priority with previous defined default per method aggregations on client configuration.<br><br> **Valid Aggregations:** `@"avg", @"count", @"sum", @"first", @"last", @"p90", @"p95", @"min", @"max"` | `NSArray*` | `@[@"avg", @"p90"]` | `@[@"last"]` | `@[@"avg", @"p90", @"count"]` |
| _agg_freq_ | Defines the aggregation frequency in **seconds** of the metric. It replaces the previous defined default per method aggregation frequency on client configuration.<br><br> **Valid Aggregation Frequencies:** `@10, @30, @60 ,@120, @180, @300` | `NSNumber*` | `@10` | `@10` | `@10` |
| _namespace_ | Defines the namespace to send with metric. It replaces the previous defined global namespace on client configuration. | `NSString*` | `@"application"` | `@"application"` | `@"application"` |
| _tags_ | Defines the tags to send with metric. It merges with priority with previous defined global tags and default per method tags on client configuration. | `NSDictionary*` | `@{}` | `@{}` | `@{@"unit": @"ms"}` |
| _timestamp_ | Defines the timestamp of the metric. This timestamp is a **POSIX/Epoch** time in **seconds**. | `NSString*` | `current timestamp` | `current timestamp` | `current timestamp` |

#### Properties

| Property | Type | Description | Access  |
|:---|:---|:---|:---|
| _logger_ | `SFLogger*` | The logger object used by client. | **readonly** |
| _isConfigValid_ | `BOOL` | A boolean value indicating whether the setted config is valid. | **readonly** |
| _isStarted_ | `BOOL` | A boolean value indicating whether the client is started. | **readonly** |

### Logger

The Logger used by Statful Client Objective-c is a simple encapsulation for the [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) logger. 

#### Class 
`SFLogger`

#### Enumerated Types

##### SFLoggerLogLevel

| Type | Description |
|:---|:---|
| `SFLoggerLogLevelError` | An enumerated type that defines the Error logger's level. This is the most restrict logger level that forces logger to only output error messages. |
| `SFLoggerLogLevelDebug` | An enumerated type that defines the Debug logger's level. This is the intermediate logger level that forces logger to output debug messages but also error messages. |
| `SFLoggerLogLevelVerbose` | An enumerated type that defines the Verbose logger's level. This is the least restrict logger level that forces logger to output all kind of messages. |

#### Methods

```objc
+ (instancetype)loggerWithDDLoggerInstance:(DDAbstractLogger <DDLogger> *)logger loggerLevel:(SFLoggerLogLevel)loggerLevel
```

This is a class method that receives a `DDAbstractLogger <DDLogger> *` and a `SFLoggerLogLevel` and returns a new `SFLogger`.

```objc
- (void)logDebug:(id)format, ...
```

This method receives a string format followed by the format params (seperated by commas) and sends a new **debug** message to be logged.

```objc
- (void)logError:(id)format, ...
```

This method receives a string format followed by the format params (seperated by commas) and sends a new **error** message to be logged.

```objc
- (void)logVerbose:(id)format, ...
```

This method receives a string format followed by the format params (seperated by commas) and sends a new **verbose** message to be logged.

#### Properties

| Property | Type | Description | Access  |
|:---|:---|:---|:---|
| _logger_ | `DDAbstractLogger <DDLogger> *` | The internal logger instance according DDLogger protocol from [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) used by SFLogger to output messages. It can be one already defined by CocoaLumberjack like `DTTYLogger`, `DDASLLogger`, `DDFileLogger` or any other custom logger object that complies with `DDAbstractLogger <DDLogger>` from CocoaLumberjack. | **readwrite** |
| _loggerLevel_ | `SFLoggerLogLevel ` | The logger level used to select which messages should be outputed. It can be  | **readwrite** |


## Still Need Help?
[//]: # (If you are feeling that you're still needing help, please visit our oficial full [Statful Documentation](http://statful.com/docs) page.)

If you are feeling that you're still needing help, please visit our oficial full Statful Documentation page.

## Authors

[Mindera - Software Craft](https://github.com/Mindera)

## License

Statful Obj-C Client is available under the MIT license. See the [LICENSE](https://raw.githubusercontent.com/statful/statful-client-objc/master/LICENSE) file for more info.
