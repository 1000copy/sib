
# 创建pod

你创建了一个库函数，想要共享它。怎么办？cocoapods可以帮忙。
 
## 准备代码
 
 首先，我们创建这样的Cocoa touch framework。添加一个swift文件（名字为：robot.swift)，内容为：

    import Foundation
    public func inc(_ i : Int)->Int{return i + 1}

编译通过即可。

## 为此代码创建podspec文件

想要CocoaPods帮忙分享，就得按照它的要求，编写一个规格文件，以便说明作者，主页，授权信息，特别是，指明源代码的位置。

现在导航到工程目录，执行如下命令：

    touch robot.podspec

粘贴内容为：

  Pod::Spec.new do |s|
    s.name             = 'robot'
    s.version          = '0.1.0'
    s.summary          = '一些屁话'
    s.description      = '一些屁话'
    s.homepage         = 'https://github.com/1000copy/robot'
    s.license          = { :type => 'MIT', :file => 'LICENSE' }
    s.author           = { '1000copy' => '1000copy@gmail.com' }
    s.source           = { :path => '~/github/pod/robot' }
    s.ios.deployment_target = '10.0'
    s.source_files = 'robot/*'
  end

其中特别重要的是s.name ,s.version , s.source ,s.source_files,没有这些信息，发布pod是不可能的。

最好执行下这个命令，检查此podspec是否正确：

    pod lib lint

## 引用pod

创建一个使用pod的Single View App工程，放到usepod内

  cd usepod
  pod init

粘贴内容到Podfile

   pod 'robot',:path=> '~/github/pod/robot/robot.podspec'

执行命令：

  pod install --verbose --no-repo-update

## 引用代码

打开usepod.workspace文件，贴入引用代码到AppDelegate.swift内：

    import UIKit
    import robot
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window : UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            print(inc(41))
            return true
        }
    }

## 发布到github仓库

文件我们会放到github仓库上，因此，一个github的账号是必要的。仓库需要创建一个release，因为podspec会引用此release。我的相关信息：

1. github账号是1000copy
2. 用于存储分享代码的仓库名字：robot

你需要在如下的命令和操作中，替代为你的对应信息。首先：

1. 创建一个仓库，命名为robot
2. 在命令行内导航到你的工程目录，执行命令以便把代码传递到github仓库内

    git init
    git add .
    git commit -m "init"
    git remote add origin <paste your URL here>
    git push -u origin master

3. 在github上，对此仓库做一个release，版本号设置为0.1.0，这个号码值会在podspec文件内引用的。

## 发布到cocoapods

执行命令发布,首先注册你的邮件，然后推送podspec。

    pod trunk register 1000copy@gmail.com
    pod trunk push robot.podspec

注意，注册邮件后，cocoapods会发送一个邮件给你，你点击里面的确认链接，即可生效。

你应该看到如下的信息反馈：

     🎉  Congrats
    
     🚀  robot (0.1.0) successfully published
     📅  August 16th, 06:03
     🌎  https://cocoapods.org/pods/robot
     👍  Tell your friends!
     

现在，你可以像其他pod一样使用robot了。



