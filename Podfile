# Uncomment the next line to define a global platform for your project
platform :ios, '11.0'
#use_modular_headers!

flutter_app_path = '../flutter_module'
newHelper_path = File.join(flutter_app_path,  '.ios', 'Flutter', 'newHelper.rb')
originHelper_path = File.join(flutter_app_path, '.ios', 'Flutter', 'podhelper.rb')
File.open(newHelper_path, 'w') { |f| f.write ""}
File.foreach(originHelper_path) { |line|
  if line.include? ":script =>"
    script_data = File.open("flutter_run_script.txt").read
    File.write(newHelper_path, "    :script => \"" + script_data + "\",\n", mode: 'a')
  else
    File.write(newHelper_path, line, mode: 'a')
  end
}
load File.join(flutter_app_path, '.ios', 'Flutter', 'newHelper.rb')

### If "<Flutter/Flutter.h> not found" error happened in build time,
### uncomment below lines and execute pod install one time
project 'MyProject', {
 "Test" => :debug,
 "Production" => :release,
}

use_frameworks!

target 'MyProject' do
  install_all_flutter_pods(flutter_app_path)
#  pod 'Flutter', :podspec => '../flutter_module/framework/Release/Flutter.podspec'
end
