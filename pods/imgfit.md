webview装入的网页，常常有幅面比较的图，这些图会超出手机的宽度，因此导致显示不完整。

比如如下案例，加入了两个图片，大小分别为：
    
    650x300
    150x150
    
在iPhone SE的模拟器下，默认情况下，前一张图会在宽度上超出，后一张可以显示完整。

    import UIKit
    class Page: UIViewController{
        var c : UIWebView!
        override func viewDidLoad() {
            super.viewDidLoad()
            c = UIWebView()
            c.frame = super.view.frame
            view.addSubview(c)
            c.frame.origin.y += 20
            let html = "<img src=\"https://via.placeholder.com/650x300\"><img src=\"https://via.placeholder.com/150x150\">"
            let url = URL(string:"http://")
            c.loadHTMLString(html, baseURL: url)
            
        }
    }
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            self.window = UIWindow(frame: UIScreen.main.bounds)
            self.window!.rootViewController = Page()
            self.window?.makeKeyAndVisible()
            return true
        }
    }

想要更加优雅的适配图片，可以使用css，强制要求显示宽度为100%,从而缩放宽度到设备宽度。做法就是加入一个html头字符串，加入到你自己装入的html字符串的头部。如：

    import UIKit
    class Page: UIViewController{
        var c : UIWebView!
        override func viewDidLoad() {
            super.viewDidLoad()
            c = UIWebView()
            c.frame = super.view.frame
            view.addSubview(c)
            c.frame.origin.y += 20
            let html = htmlhead + "<img src=\"https://via.placeholder.com/650x300\"><img src=\"https://via.placeholder.com/150x150\">"
            let url = URL(string:"http://")
            c.loadHTMLString(html, baseURL: url)
            
        }
    }
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            self.window = UIWindow(frame: UIScreen.main.bounds)
            self.window!.rootViewController = Page()
            self.window?.makeKeyAndVisible()
            return true
        }
    }
    let htmlhead =  "<!DOCTYPE html>" +
        "<html>" +
        "<head>" +
        "<meta charset=\"UTF-8\">" +
        "<style type=\"text/css\">" +
        "html{margin:0;padding:0;}" +
        "body {" +
        "margin: 0;" +
        "padding: 0;" +
        "}" +
        "img{" +
        "width: 90%;" +
        "height: auto;" +
        "display: block;" +
        "margin-left: auto;" +
        "margin-right: auto;" +
        "}" +
        "</style>" +
    "</head>"
    
出来的效果，是两张图片全部充满宽度，其中一个图片缩小，不会产生锯齿，一个图片放大，有些锯齿。对于大量照片的html文档，一般都是缩小的，因此看起来还是过得去的。

