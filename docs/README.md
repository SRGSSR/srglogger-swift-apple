[![SRG Logger logo](README-images/logo.png)](https://github.com/SRGSSR/srglogger-swift-apple)

[![GitHub releases](https://img.shields.io/github/v/release/SRGSSR/srglogger-swift-apple)](https://github.com/SRGSSR/srglogger-swift-apple/releases) [![platform](https://img.shields.io/badge/platfom-ios%20%7C%20tvos%20%7C%20watchos-blue)](https://github.com/SRGSSR/srglogger-swift-apple) [![Build Status](https://travis-ci.org/SRGSSR/srglogger-swift-apple.svg?branch=master)](https://travis-ci.org/SRGSSR/srglogger-swift-apple/branches) [![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage) [![GitHub license](https://img.shields.io/github/license/SRGSSR/srglogger-swift-apple)](https://github.com/SRGSSR/srglogger-swift-apple/blob/master/LICENSE) 

## About

The SRG Logger library provides a simple way to unify logging between SRG SSR libraries and applications, but can be used by any other application or library as well.

Five logging levels are available, which should match most needs:

* `Verbose` for detailed technical information.
* `Debug` for debugging information.
* `Info` for information that may be helpful for troubleshooting errors.
* `Warning` for information about conditions which might lead to a failure.
* `Error` for information about failures.

The library automatically bridges with standard logging frameworks, in the following order:

* If your project uses [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack), messages will be forwarded to it.
* If your project can use [Apple unified logging](https://developer.apple.com/reference/os/1891852-logging), available starting from iOS 10, tvOS 10 and watchOS 3, messages will be forwarded to the system console.

If neither is available, no logging will take place. You can install a verbose `NSLog` based logger, provided as well, if you need a quick way to setup logging (this logger logs all messages and can slow down your application).

The library does not provide any logging to [NSLogger](https://github.com/fpillet/NSLogger). You can namely automatically bridge CocoaLumberjack into NSLogger using [Cédric Luthi's interface](https://github.com/0xced/XCDLumberjackNSLogger) between them.

## Compatibility

The library is suitable for applications running on iOS 9, tvOS 12, watch OS 5 and above. The project is meant to be opened with the latest Xcode version and built with the Swift 5 compiler.

Objective-C projects must use the [SRG Logger framework](https://github.com/SRGSSR/srglogger-ios) directly.

## Contributing

If you want to contribute to the project, have a look at our [contributing guide](CONTRIBUTING.md).

## Installation

The library can be added to a project using [Carthage](https://github.com/Carthage/Carthage) by adding the following dependency to your `Cartfile`:
    
```
github "SRGSSR/srglogger-swift-apple"
```

For more information about Carthage and its use, refer to the [official documentation](https://github.com/Carthage/Carthage).

### Dependencies

The library requires the following frameworks to be added to any target requiring it:

* `SRGLogger`: The main library framework.
* `SRGLogger_Swift`: The Swift code companion framework.

### Dynamic framework integration

1. Run `carthage update` to update the dependencies (which is equivalent to `carthage update --configuration Release`). 
2. Add the frameworks listed above and generated in the `Carthage/Build/(iOS|tvOS|watchOS)` folder to your target _Embedded binaries_.

If your target is building an application, a few more steps are required:

1. Add a _Run script_ build phase to your target, with `/usr/local/bin/carthage copy-frameworks` as command.
2. Add each of the required frameworks above as input file `$(SRCROOT)/Carthage/Build/(iOS|tvOS|watchOS)/FrameworkName.framework`.

### Static framework integration

1. Run `carthage update --configuration Release-static` to update the dependencies. 
2. Add the frameworks listed above and generated in the `Carthage/Build/(iOS|tvOS|watchOS)/Static` folder to the _Linked frameworks and libraries_ list of your target.
3. Also add any resource bundle `.bundle` found within the `.framework` folders to your target directly.
4. Add the `-all_load` flag to your target _Other linker flags_.

## Usage

When you want to use classes or functions provided by the library in your code, you must import it from your source files first:

```swift
import SRGLogger_Swift
```

### Logging messages

To log a message, simply call the function corresponding to the desired level:

```Swift
SRGLogInfo(subsystem: "com.myapp", category: "Weather", message: "The temperature is \(temperature)")
```

You can provide two optional arguments when logging a message:

* A subsystem, here `com.myapp`, which identifies the library or application you log from.
* A category, here `Weather`, which identifies which part of the code the log is related to.

To avoid specifiying the subsystem in your application or library each time you call the logging function, you can define your own set of helpers which always set this value consistently, for example:

```swift
func MyAppLogVerbose(category : String?, message: String, file: String = #file, function: String = #function, line: UInt = #line) {
    SRGLogVerbose(subsystem: "com.myapp", category: category, message: message, file: file, function: function, line: line);
}

func MyAppLogDebug(category : String?, message: String, file: String = #file, function: String = #function, line: UInt = #line) {
    SRGLogDebug(subsystem: "com.myapp", category: category, message: message, file: file, function: function, line: line);
}

func MyAppLogInfo(category : String?, message: String, file: String = #file, function: String = #function, line: UInt = #line) {
    SRGLogInfo(subsystem: "com.myapp", category: category, message: message, file: file, function: function, line: line);
}

func MyAppLogWarning(category : String?, message: String, file: String = #file, function: String = #function, line: UInt = #line) {
    SRGLogWarning(subsystem: "com.myapp", category: category, message: message, file: file, function: function, line: line);
}

func MyAppLogError(category : String?, message: String, file: String = #file, function: String = #function, line: UInt = #line) {
    SRGLogError(subsystem: "com.myapp", category: category, message: message, file: file, function: function, line: line);
}
```

### Interfacing with other loggers

If the default log handler does not suit your needs (or if you simply want to inhibit logging), set a new handler to forward the messages and contextual information to your other logger:

```swift
SRGLogger.setLogHandler { (message, level, subsystem, category, file, function, line) in
    // Foward information to another logger
}
```

## Building the project

A [Makefile](../Makefile) provides several targets to build and package the library. The available targets can be listed by running the following command from the project root folder:

```
make help
```

Alternatively, you can of course open the project with Xcode and use the available schemes.

## Thread-safety

Logging can be performed from any thread.

## Apple unified logging troubleshooting

If you are using Apple unified logging and do not see the logs:

1. Check that the scheme you use does not have the `OS_ACTIVITY_MODE` environment variable set to `disable`, or `OS_ACTIVITY_DT_MODE` set to `NO`.
1. If you do not see lower level logs in the `Console.app`, ensure that the items _Include Info Messages_ and `Include Debug Messages` are checked in the console `Action` menu. If these options are grayed out, try updating the logging configuration for your subsystem first by running the following command from a terminal:

	```
	$ sudo log config --mode "level:debug" --subsystem <subsystem>
	```
	
1. Use the search to locate entries for your application name, and right-click on an entry to filter by application, subsystem or category
1. Read the [official documentation](https://developer.apple.com/documentation/os/logging) if you still have issues

## Credits

This logger implementation is heavily based on [a Cédric Luthi's Stack Overflow post](http://stackoverflow.com/questions/34732814/how-should-i-handle-logs-in-an-objective-c-library/34732815#).

## License

See the [LICENSE](../LICENSE) file for more information.
