
# 创建pod

你创建了一个迷幻的View，想要向全世界共享它。怎么办？cocoapods可以帮忙。
 
 ##创建一个工程，其中有你需要分享的代码
 
 首先，我们创建这样的一个Single View App。添加一个swift文件（名字为：FantasticView.swift)，内容为：
 ```swift
import UIKit
class FantasticView : UIView {
    let colors : [UIColor] = [.red, .orange, .yellow, .green, .blue, .purple]
    var colorCounter = 0
    override init(frame: CGRect) {
        super.init(frame: frame)
        let scheduledColorChanged = Timer.scheduledTimer(withTimeInterval: 2.0, repeats: true) { (timer) in  //1
            UIView.animate(withDuration: 2.0) {  //2
                self.layer.backgroundColor = self.colors[self.colorCounter % 6].cgColor  //3
                self.colorCounter+=1  //4
            }
        }
        scheduledColorChanged.fire()
        // The Main Stuff
    }
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        // You don't need to implement this
    }
}
```
然后编写代码，做一个demo，把这个视图用起来，代码覆盖ViewController.swift:
 ```swift
import UIKit
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        let fantasticView = FantasticView(frame: self.view.bounds)
        self.view.addSubview(fantasticView)
    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
 ```
 
跑起来后，你可以看到一个变化颜色的视图，说明类FantasticView正常工作了。这个文件，就是我们想要分享的。

接下来，如果想要CocoaPods帮忙，给创建一个文件.podspec文件，并把这个文件传递到CocoaPods网站上。此文件会告诉想要使用此pod的人如下信息：
    
    1. 这个pod叫什么名字
    2. 版本号多少
    3. 代码包括哪些文件
    4. 文件在哪里

等等必要的和描述性的信息。

## 创建一个github仓库，推送文件，并做一个release

文件我们会放到github仓库上，因此，一个github的账号是必要的。仓库需要创建一个release，因为podspec会引用此release。我的相关信息：

1. github账号是1000copy
2. 用于存储分享代码的仓库名字：FantasticView

你需要在如下的命令和操作中，替代为你的对应信息。首先
1. 创建一个仓库，命名为FantasticView
2. 在命令行内导航到你的工程目录，执行命令以便把代码传递到github仓库内

    git init
    git add .
    git commit -m "init"
    git remote add origin <paste your URL here>
    git push -u origin master
3. 在github上，对此仓库做一个release，版本号设置为0.1.0，这个号码值会在podspec文件内引用的。

## 终于，我们可以创建一个pod了。

依然需要在命令行内，导航到你的工程目录，然后：

    touch FantasticView.podspec

粘贴内容为：

    Pod::Spec.new do |s|
      s.name             = 'FantasticView'
      s.version          = '0.1.0'
      s.summary          = '一些屁话'
      s.description      = <<-DESC
    更多行
    行的
    屁话
                           DESC
     
      s.homepage         = 'https://github.com/1000copy/FantasticView'
      s.license          = { :type => 'MIT', :file => 'LICENSE' }
      s.author           = { '1000copy' => '1000copy@gmail.com' }
      s.source           = { :git => 'https://github.com/1000copy/FantasticView.git', :tag => s.version.to_s }
      s.ios.deployment_target = '10.0'
      s.source_files = 'FantasticView/*'
    end
    
其中特别重要的是s.name ,s.version , s.source ,s.source_files,没有这些信息，发布pod是不可能的。

接着执行命令，检查此podspec是否正确：

    pod lib lint
    
## 发布

执行命令发布,首先注册你的邮件，然后推送podspec。

    pod trunk register 1000copy@gmail.com
    pod trunk push FantasticView.podspec
注意，注册邮件后，cocoapods会发送一个邮件给你，你点击里面的确认链接，即可生效。

你应该看到如下的信息反馈：

     🎉  Congrats
    
     🚀  FantasticView (0.1.0) successfully published
     📅  August 16th, 06:03
     🌎  https://cocoapods.org/pods/FantasticView
     👍  Tell your friends!
     

现在，你可以像其他pod一样使用FantasticView了。

意译自：http://www.appcoda.com/cocoapods-making-guide/

