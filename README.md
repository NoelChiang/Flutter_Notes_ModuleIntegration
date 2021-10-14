# Flutter_Notes_ModuleIntegration
Keynotes of integrating flutter module project into existed iOS project

### Environment & Tool
Flutter SDK 2.5.1  
Dart 2.14.2  
Xcode 12.4  
Swift 5  
iOS 14.x

### Flow
##### 1. Create flutter project

```
$flutter create --template module {my_flutter_module_name}
```

##### 2. Import module into existed iOS project
- ###### Option.A, Integrate flutter source code and library through cocoapods  

  Before integration, execute below command will download relative flutter package and setup ios runner
  
  ```
  flutter pub get
  ```
  
  At below path, podhelper.rb file will be generated automatically
  
  ```
  {Flutter_module}/.ios/Flutter/podhelper.rb
  ```
  
  Load this above file in Podfile then execute pod install,
  
  ```
  flutter_module_path = '{Flutter_module}'
  load File.join(flutter_module_path, '.ios', 'Flutter', 'podhelper.rb')
  
  target {Target_name} do
  install_all_flutter_pods(flutter_module_path)
  ```
- ###### Option.B, Integrate my flutter module in framework manually, and install flutter library through cocoapods 
  
  Export flutter module to framework file
  
  ```
  flutter build ios-framework --cocoapods --output={Output_file_path}
  ```
  
  Drag all framework files to xcode project > Build Phrases > Link Binary With Libraries  
  Set dynamic frameworks to Embed & Sign in xcode project > General > Frameworks, Libraries and Embeded Contents  
  *App.xcframework is static framework, keep it as Do Not Embed*  

  In Podfile, add below phrases and pod install, (Release can change to Debug or Profile based on purpose)
  
  ```
  target {Target_name} do
  pod 'Flutter', :podspec => '{Output_file_path/Release/Flutter.podspec}'
  ```
##### 3. Use flutter component in iOS project
Create flutter engine with default entry point: main()

```
let engine = FlutterEngine(name: "{Engine_name}")
engine.run()
GeneratedPluginRegistrant.register(engine)
```

Or run engine with designated entry point

```
let engine = FlutterEngine(name: "engine_name")
engine.run(withEntryPoint: "{Entry_point}")
GeneratedPluginRegistrant.register(engine)
```

Before run engine with entry point, a decoration needs to be added in flutter source code

```
@progma('vm: entry-point')    // Decoration
void Entry_point() => runApp()
```

Create FlutterViewController through engine

```
let flutterVC = FlutterViewController(engine: engine, nibName: nil, bundle: nil)
```

FlutterViewController can be used as UIViewController to present
If data transfer is needed, use FlutterViewController to generate method channel

```
let channel = FlutterMethodChannel(name: {Channel_name}, binaryMessenger: flutterVC as! FlutterBinaryMessenger)
```

Send data by invokeMethod, receive data by setMethodCallHandler

```
channel.invokeMethod('{Method_name}', {Arguments})
channel.setMethodCallHandler { (call, result) in 
  // Statements
}
```

