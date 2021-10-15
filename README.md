# Flutter_Notes_ModuleIntegration
Keynotes of integrating flutter module project into existed iOS project

## Environment & Tool
Flutter SDK 2.5.1  
Dart 2.14.2  
Xcode 12.4  
Swift 5  
iOS 14.x

## Flow
#### 1. Create flutter project

```
$flutter create --template module {my_flutter_module_name}
```

#### 2. Import module into existed iOS project
- ##### Option.A, Integrate flutter source code and library through cocoapods  

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
- ##### Option.B, Integrate my flutter module in framework manually, and install flutter library through cocoapods 
  
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
#### 3. Use flutter component in iOS project
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
#### 4. Present flutter component in iOS project
- As ViewController: present directly
  
  ```
  // In UIViewController which will present flutterVC
  self.present(flutterVC, animated: true, completion: nil)
  ```
  
- As View: put flutterVC into container view 
  
  ```
  // In UIViewController which container view will added to
  let container = UIView(frame: CGRect(x:0, y:0, width: 100, height: 100))
  container.addSubview(flutterVC.view)
  NSLayoutConstraint.activate([
      flutterVC.view.leadingAnchor.constraint(equalTo: container.leadingAnchor),
      flutterVC.view.trailingAnchor.constraint(equalTo: container.trailingAnchor),
      flutterVC.view.topAnchor.constraint(equalTo: container.topAnchor),
      flutterVC.view.bottomAnchor.constraint(equalTo: container.bottomAnchor),
  ])
  flutterVC.didMove(toParent: self)
  self.view.addSubview(container)
  ```

#### 5. Data exchange between native and flutter

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


## Trouble shooting
#### 1. Build simulator failed
If `<Flutter/Flutter.h> not found` was happened, add a map of build configuration in Podfile might solve the issue. 
Basically flutter only support build configuration as 'debug', 'profile' and 'release' 

```
project '{Project_name}', {
  "Test" => :debug,
  "Production" => :release,
}
```

#### 2. Build failed with log "Unknown FLUTTER_BUILD_MODE=xxx"
When integration through option.A, a function `install_flutter_application_pod` in `podhelper.rb`  
will add a run script into xcode > Build Phrases, below is the snippet

```
script_phase :name => 'Run Flutter Build main_module Script',
    :script => "set -e\nset -u\nsource \"#{flutter_export_environment_path}\"\nexport VERBOSE_SCRIPT_LOGGING=1 && \"$FLUTTER_ROOT\"/packages/flutter_tools/bin/xcode_backend.sh build",
    :execution_position => :before_compile
```
This run script will refer to file `xcode_backend.sh` in flutter sdk folder to do build configuration mapping.  
If xcode's build configuration did not contain 'debug', 'profile', or 'release' in their name, exception will be raised.  

Rename build configuration can solve this problem,  
if they can't be changed, export FLUTTER_BUILD_MODE = 'debug'/'profile'/'release' in build phases' script can be another solution

```
set -e
set -u
source "${SRCROOT}/../{Flutter_module_path}/.ios/Flutter/flutter_export_environment.sh"

// ===== Insert below code 
if [[ "${PLATFORM_DISPLAY_NAME}" == *"Simulator"* ]]; then
	mode="debug"
else
	mode="release"
fi
export FLUTTER_BUILD_MODE=$mode
// =====

export VERBOSE_SCRIPT_LOGGING=1 && "$FLUTTER_ROOT"/packages/flutter_tools/bin/xcode_backend.sh build
```

In Podfile, add code to replace the insert run script in podhelper.rb,  
all things can be done by pod install

```
flutter_app_path = '{Flutter_module_path}'
newHelper_path = File.join(flutter_app_path,  '.ios', 'Flutter', 'newHelper.rb')
originHelper_path = File.join(flutter_app_path, '.ios', 'Flutter', 'podhelper.rb')
File.open(newHelper_path, 'w') { |f| f.write ""}
File.foreach(originHelper_path) { |line|
  if line.include? ":script =>"
  
    // Replace run script with pre defined script.txt
    script_data = File.open("flutter_run_script.txt").read
    File.write(newHelper_path, "    :script => \"" + script_data + "\",\n", mode: 'a')
    
  else
    File.write(newHelper_path, line, mode: 'a')
  end
}
load File.join(flutter_app_path, '.ios', 'Flutter', 'newHelper.rb')
```

#### 3. In develping, revised flutter source doesn't reflect to project 
If revised flutter module doesn't reflect to project, try below flow

```
${Flutter_module_folder} flutter clean
${Flutter_module_folder} flutter pub get

${Project_folder} pod install
```
