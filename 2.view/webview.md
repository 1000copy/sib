
## WebView

UIWebView用来在App内嵌入Web页面。如下代码装入Apple.com官方首页在App内:

    import UIKit
    class Page: UIViewController{
        var c : UIWebView!
        override func viewDidLoad() {
            super.viewDidLoad()
            c = UIWebView()
            c.frame = super.view.frame
            view.addSubview(c)
            c.frame.origin.y += 20
            let url = URL(string:"http://apple.com")
            let ro = URLRequest(url:url!)
            c.loadRequest(ro)
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

关键代码在于：

    let url = URL(string:"http://apple.com")
    let ro = URLRequest(url:url!)
    c.loadRequest(ro)

构建URL对象和URLRequest请求对象，然后使用WebView的方法loadRequest装入请求，即可加载页面在WebView内。


## 在webview的当前网页上提取信息的方法

使用UIWebView装载一个网页后，可能需要提取其内的信息，比较好的方法是使用JavaScript。方法UIWebView.stringByEvaluatingJavaScript可以执行一个脚本。

### 提取页面信息

执行如下代码，点击页面左上方的run js按钮，可以显示一个对话框，内容为当前网页的标题（title）：

    import UIKit
    class Page: UIViewController{
        var c : UIWebView!
        override func viewDidLoad() {
            super.viewDidLoad()
            c = UIWebView()
            c.frame = super.view.frame
            view.addSubview(c)
            c.frame.origin.y += 100
            c.frame.size.height = 300
            let url = URL(string:"http://apple.com")
            let ro = URLRequest(url:url!)
            c.loadRequest(ro)
            let button = UIButton()
            button.setTitle("run js", for: .normal)
            button.addTarget(self, action: #selector(tap), for: .touchDown)
            button.frame = CGRect(x: 0, y: 70, width: 100, height: 20)
            view.addSubview(button)
        }
        func tap(){
            c.stringByEvaluatingJavaScript(from: "function showtitle(){alert(document.title)};showtitle()")
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

如果javas使用的函数或者模块比较大，可以把它们放到一个文件内，作为资源包含，启动时加载并求值。这样是很方便的。接下来的案例，把上面案例的js函数showtitle()作为资源文件。首先func tap内修改为从资源中加载：

        func tap(){
            let jsCode = try! String(contentsOf: URL(fileURLWithPath: Bundle.main.path(forResource: "pack", ofType: "js")!))
            c.stringByEvaluatingJavaScript(from: jsCode)
            c.stringByEvaluatingJavaScript(from: "showtitle()")
        }
其次，创建一个资源文件，名为pack.js ,内容为：

      function showtitle(){alert(document.title)};

代码运行后，点击run js按钮，效果和前一个案例相同。

### 提取图片信息

有时候，需要截获WebView的手势操作到当前代码内，在此代码中获取当前手势触控的位置上的元素。这个场景下，就可以把js模块代码放到资源文件内，在触控代码中对此js调用和求值，获得它的输出：

    import UIKit
    class Page: UIViewController,UIGestureRecognizerDelegate{
        var c : UIWebView!
        var tapGesture : UITapGestureRecognizer!
        override func viewDidLoad() {
            super.viewDidLoad()
            c = UIWebView()
            c.frame = super.view.frame
            view.addSubview(c)
            c.frame.origin.y += 100
            c.frame.size.height = 300
            let url = URL(string:"https://httpbin.org/image/png")//must be a https url ,otherwise iOS will fobidden it 
            let ro = URLRequest(url:url!)
            c.loadRequest(ro)
            let button = UIButton()
            button.setTitle("run js", for: .normal)
            button.addTarget(self, action: #selector(tap), for: .touchDown)
            button.frame = CGRect(x: 0, y: 70, width: 100, height: 20)
            view.addSubview(button)
            tapGesture = UITapGestureRecognizer(target: self, action:#selector(tapHandler(_:)))
            self.tapGesture!.delegate = self
            self.c.addGestureRecognizer(self.tapGesture!);
        }
        func tap(){
            let jsCode = try! String(contentsOf: URL(fileURLWithPath: Bundle.main.path(forResource: "pack", ofType: "js")!))
            c.stringByEvaluatingJavaScript(from: jsCode)
            c.stringByEvaluatingJavaScript(from: "showtitle()")
        }
        func gestureRecognizer(_: UIGestureRecognizer,  shouldRecognizeSimultaneouslyWith shouldRecognizeSimultaneouslyWithGestureRecognizer:UIGestureRecognizer) -> Bool
        {
            return true
        }
        func tapHandler(_ tap :UITapGestureRecognizer){
            let tapPoint = tap.location(in: tap.view)
            print(tapPoint)
            let script = String(format: "getHTMLElementAtPoint(%i,%i)", Int(tapPoint.x),Int(tapPoint.y))
            let imgSrc = self.c.stringByEvaluatingJavaScript(from: script)
            print(imgSrc)
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
执行代码，点击页面的图片，就可以看到控制台上的输出：

    (168.0, 65.0)
    Optional("https://httpbin.org/image/png,100,100,110,0")

委托方法 gestureRecognizer(_:shouldRecognizeSimultaneouslyWith)是非常必要的，没有它的话，WebView会自己处理手势，而不是转移控制器给tapHandler(_:)。


## WebView 显示SVG

SVG文件是矢量图标准之一，特点是可以缩放，并且可以用可以阅读的源代码的方式（而不是二进制）来存储图形信息。比如如下文件就是一个svg文件：

    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 350 100">
      <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="7" 
        refX="0" refY="3.5" orient="auto">
          <polygon points="0 0, 10 3.5, 0 7" />
        </marker>
      </defs>
      <line x1="0" y1="50" x2="250" y2="50" stroke="#000" 
      stroke-width="8" marker-end="url(#arrowhead)" />
    </svg>

它是一个箭头图。可以使用UIWebView视图加载此文件并显示。首先把SVG文件作为资源文件加入工程，命名为1.svg。

其实，使用如下代码加载此文件：

        var path: String = Bundle.main.path(forResource: "1", ofType: "svg")!
        var url: NSURL = NSURL.fileURL(withPath: path) as NSURL
        var request: NSURLRequest = NSURLRequest(url: url as URL)
        webview?.loadRequest(request as URLRequest)

完整代码如下：

      import UIKit
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            self.window = UIWindow(frame: UIScreen.main.bounds)
            let page = Page()
            page.view.backgroundColor = .blue
            self.window!.rootViewController = page
            self.window?.makeKeyAndVisible()
            return true
        }
    }
    class Page: UIViewController {
        var count = 0
        var webview : UIWebView!
        var webview1 : UIWebView!
        override func viewDidLoad() {
            super.viewDidLoad()
            webview = UIWebView()
            webview?.frame = CGRect(x: 0, y: 100, width: 100, height: 100)
            view.addSubview(webview!)
            webview1 = UIWebView()
            webview1.frame = CGRect(x: 0, y: 200, width: 50, height: 50)
            view.addSubview(webview1!)
            let path: String = Bundle.main.path(forResource: "1", ofType: "svg")!
            let url: NSURL = NSURL.fileURL(withPath: path) as NSURL
            let request: NSURLRequest = NSURLRequest(url: url as URL)
            webview?.loadRequest(request as URLRequest)
            webview1.loadRequest(request as URLRequest)
        }
    }

本意是想用它来替换位图图标，但是看起来加载速度堪忧。

## 图片适配的一种方法

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

