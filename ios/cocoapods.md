# Cocoapods

CocoaPods is a dependency manager for Swift and Objective-C Cocoa projects

# Install

install Ruby gem cocoa pods
```shell
sudo gem install cocoapods
```

install Macbook M1 Apple Silicone
install [brew](https://docs.brew.sh/Installation) 
```shell
brew install cocoapods
```

# Podfile

first create Xcode project `testapp`

create file `Podfile`
```ruby
platform :ios, '13.0'

target 'inst' do
  use_frameworks!

  pod 'Firebase/Analytics'

  target 'instTests' do
    inherit! :search_paths
  end

  target 'instUITests' do

  end

end
```

# Using

Install pods specified in `Podfile`
```shell
pod install
```

Open with Xcode file `project.xcworkspace` and not `project.xcodeproj`

# References
1. [original documentation](https://cocoapods.org)
2. [cocoapods guides](https://guides.cocoapods.org/using/using-cocoapods.html)
