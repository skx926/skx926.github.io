---
title: 使用Trunk发布开源库到Cocoapods
date: 2017-05-16 16:49:20
tags: iOS开发
---

### 注册

##### 安装Cocoapods

```bash
$ sudo gem install cocoapods
$ pod setup
```

`pod setup` 的过程就是把 [Cocoapods/Specs](https://github.com/CocoaPods/Specs.git) 克隆到本地的过程，由于这个库包含了所有发布到
Cocoapods的开源库的说明，现在已经有数百兆的大小了，而且由于某众所周知的原因，这个过程会比较慢，需要耐心等待。

##### 注册Trunk

```bash
$ pod trunk register skx926@gmail.com 'Kyle Sun' --description='iMac' --verbose
```

这个是我注册时用的命令，使用时需要把邮箱和名字以及描述修改成你的。这个描述是用来区分不同的设备的，如果你有多台开发设备，它就可以说明你是使用哪一台设备进行的操作。

注册成功之后Cocoapods会给你发送邮件，点击邮件中的链接进行验证就注册成功了，可以使用 `pod trunk me` 来查看自己的注册信息：

```bash
$ pod trunk me
  - Name:     Kyle Sun
  - Email:    skx926@gmail.com
  - Since:    January 3rd, 03:28
  - Pods:
    - KSPhotoBrowser
    - KSGuideController
  - Sessions:
    - January 3rd, 03:28 - September 19th, 20:57. IP: 113.87.161.243
    Description: skx926
    - January 3rd, 08:29 -       May 19th, 18:01. IP: 36.36.186.238 
    Description: hackintosh
    - May 19th, 22:45    - September 24th, 22:45. IP: 36.36.187.9   
    Description: iMac
```

### 部署你的Pod

首先需要在项目目录下建立一个 `podspec` 文件：

```ruby
#
#  Be sure to run `pod spec lint KSGuideController.podspec' to ensure this is a
#  valid spec and to remove all comments including this before submitting the spec.
#
#  To learn more about Podspec attributes see http://docs.cocoapods.org/specification.html
#  To see working Podspecs in the CocoaPods repo see https://github.com/CocoaPods/Specs/
#

Pod::Spec.new do |s|

  # ―――  Spec Metadata  ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  These will help people to find your library, and whilst it
  #  can feel like a chore to fill in it's definitely to your advantage. The
  #  summary should be tweet-length, and the description more in depth.
  #

  s.name         = "KSGuideController"
  s.version      = "0.0.1"
  s.summary      = "A beautiful animated novice guide controller."

  # This description is used to generate tags and improve search results.
  #   * Think: What does it do? Why did you write it? What is the focus?
  #   * Try to keep it short, snappy and to the point.
  #   * Write the description between the DESC delimiters below.
  #   * Finally, don't worry about the indent, CocoaPods strips it!
  # s.description  = <<-DESC
  #               DESC

  s.homepage     = "https://github.com/skx926/KSGuideController"
  # s.screenshots  = "www.example.com/screenshots_1.gif", "www.example.com/screenshots_2.gif"


  # ―――  Spec License  ――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Licensing your code is important. See http://choosealicense.com for more info.
  #  CocoaPods will detect a license file if there is a named LICENSE*
  #  Popular ones are 'MIT', 'BSD' and 'Apache License, Version 2.0'.
  #

  s.license      = "MIT"
  # s.license      = { :type => "MIT", :file => "FILE_LICENSE" }


  # ――― Author Metadata  ――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Specify the authors of the library, with email addresses. Email addresses
  #  of the authors are extracted from the SCM log. E.g. $ git log. CocoaPods also
  #  accepts just a name if you'd rather not provide an email address.
  #
  #  Specify a social_media_url where others can refer to, for example a twitter
  #  profile URL.
  #

  s.author             = { "Kyle Sun" => "skx926@gmail.com" }
  # Or just: s.author    = "Kyle Sun"
  # s.authors            = { "Kyle Sun" => "skx926@gmail.com" }
  s.social_media_url   = "https://twitter.com/skx926"

  # ――― Platform Specifics ――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  If this Pod runs only on iOS or OS X, then specify the platform and
  #  the deployment target. You can optionally include the target after the platform.
  #

  # s.platform     = :ios
  s.platform     = :ios, "8.0"

  #  When using multiple platforms
  # s.ios.deployment_target = "5.0"
  # s.osx.deployment_target = "10.7"
  # s.watchos.deployment_target = "2.0"
  # s.tvos.deployment_target = "9.0"


  # ――― Source Location ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Specify the location from where the source should be retrieved.
  #  Supports git, hg, bzr, svn and HTTP.
  #

  s.source       = { :git => "https://github.com/skx926/KSGuideController.git", :tag => "#{s.version}" }


  # ――― Source Code ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  CocoaPods is smart about how it includes source code. For source files
  #  giving a folder will include any swift, h, m, mm, c & cpp files.
  #  For header files it will include any header in the folder.
  #  Not including the public_header_files will make all headers public.
  #

  s.source_files  = "KSGuideController", "KSGuideController/**/*.{swift}"
  # s.exclude_files = "Classes/Exclude"

  # s.public_header_files = "KSGuideController/**/*.h"


  # ――― Resources ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  A list of resources included with the Pod. These are copied into the
  #  target bundle with a build phase script. Anything else will be cleaned.
  #  You can preserve files from being cleaned, please don't preserve
  #  non-essential files like tests, examples and documentation.
  #

  # s.resource  = "icon.png"
  s.resources = "KSGuideController/Resources/*.png"

  # s.preserve_paths = "FilesToSave", "MoreFilesToSave"


  # ――― Project Linking ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Link your library with frameworks, or libraries. Libraries do not include
  #  the lib prefix of their name.
  #

  s.framework  = "UIKit"
  # s.frameworks = "SomeFramework", "AnotherFramework"

  # s.library   = "iconv"
  # s.libraries = "iconv", "xml2"


  # ――― Project Settings ――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  If your library depends on compiler flags you can set them in the xcconfig hash
  #  where they will only apply to your library. If you depend on other Podspecs
  #  you can include multiple dependencies to ensure it works.

  # s.requires_arc = true

  # s.xcconfig = { "HEADER_SEARCH_PATHS" => "$(SDKROOT)/usr/include/libxml2" }
  # s.dependency "YYWebImage", "~> 1.0.5"

end
```

上面的文件里每个项目的意思我就不赘述了，里面的注释写的都很清楚，你需要用的配置项就把前面的 `#` 号去掉。

`trunk` 会根据这个文件里所描述的内容来对你的项目进行验证，所以在验证之前你需要把你的项目先 `push` 到 Github 上，同时给这次的 `commit` 添加一个 `tag`，这个 `tag` 需要和 `s.version` 相同，这样别人在集成你的库的时候才会根据版本号下载到相对应的 `commit`。

```bash
$ git add -A
$ git commit -m "Release 0.0.1"
$ git push --tags
$ git push origin master
```

然后在项目目录下运行下面的命令来对你的 `podspec` 文件进行验证并上传到 [Cocoapods/Specs](https://github.com/CocoaPods/Specs.git)：

```bash
pod trunk push KSGuideController.podspec --allow-warnings
```

验证的条件比较严格，一旦有警告也无法通过，但是可以通过添加 `--allow-warnings` 参数来忽略警告。部署成功之后Cocoapods会在Twiter上@你。这时你再运行 `pod setup` 来更新本地的Pods仓库，然后使用 `pod search KSGuideController` 来搜索你刚发布的项目：

```bash
$ pod search KSGuideController
-> KSGuideController (0.0.1)
   A beautiful animated novice guide controller.
   pod 'KSGuideController', '~> 0.0.1'
   - Homepage: https://github.com/skx926/KSGuideController
   - Source:   https://github.com/skx926/KSGuideController.git
   - Versions: 0.0.1 [master repo]
```

大功告成，这样别人就可以在项目中使用Cocoapods集成你的开源库了！