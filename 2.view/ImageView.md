
## ImageView

UIImageView可以显示一张图、或者一组动图。

如下案例代码可以显示一张图片：

    import UIKit
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
    extension UIImage {
        class func imageWithColor(_ color: UIColor) -> UIImage {
            let rect = CGRect(x: 0.0, y: 0.0, width: 10.0,height: 10.0 )
            UIGraphicsBeginImageContext(rect.size)
            let context = UIGraphicsGetCurrentContext()
            context?.setFillColor(color.cgColor)
            context?.fill(rect)
            let image = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()               
            return image!
        }
    }
    class Page: UIViewController {
        var bgImage: UIImageView!
        override func viewDidLoad() {
            super.viewDidLoad()
            bgImage = UIImageView()
            bgImage!.frame = CGRect(x: 0,y: 0,width: 100,height: 100)
            bgImage.image = UIImage.imageWithColor(UIColor.red)
            self.view.addSubview(bgImage!)
        }
    }

需要做的就是实例化UIImageView后，把属性.image赋值为UIImage的实例，后者可以用来装入图片。这里再次使用UIImage的扩展函数：

    class func imageWithColor(_ color: UIColor) -> UIImage 

现场生产一张指定颜色填充的图片，通过传递不同的颜色值，就可以得到不同颜色的小方块图片对象。

如果想要显示一组动图，可以这样写代码：

    import UIKit
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
    extension UIImage {
        class func imageWithColor(_ color: UIColor) -> UIImage {
            let rect = CGRect(x: 0.0, y: 0.0, width: 10.0,height: 10.0 )
            UIGraphicsBeginImageContext(rect.size)
            let context = UIGraphicsGetCurrentContext()
            context?.setFillColor(color.cgColor)
            context?.fill(rect)
            let image = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()               
            return image!
        }
    }
    class Page: UIViewController {
        var bgImage: UIImageView!
        override func viewDidLoad() {
            super.viewDidLoad()
            bgImage = UIImageView()
            bgImage.animationImages = [UIImage.imageWithColor(UIColor.red),
                                       UIImage.imageWithColor(UIColor.yellow),
                                       UIImage.imageWithColor(UIColor.black),
                                       UIImage.imageWithColor(UIColor.blue)]
            bgImage!.frame = CGRect(x: 0,y: 0,width: 100,height: 100)
            self.view.addSubview(bgImage!)
            bgImage.animationDuration = 1
            bgImage.startAnimating()
        }
        override func viewDidDisappear(_ animated: Bool) {
            bgImage.stopAnimating()
        }
    }

运行代码时，你可以看到一个矩形区域正在切换4个图片的显示，每个图片是一种颜色。

UIImageView可以响应事件，如下代码演示轻按事件：

    import UIKit
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
    extension UIImage {
        class func imageWithColor(_ color: UIColor) -> UIImage {
            let rect = CGRect(x: 0.0, y: 0.0, width: 10.0,height: 10.0 )
            UIGraphicsBeginImageContext(rect.size)
            let context = UIGraphicsGetCurrentContext()
            context?.setFillColor(color.cgColor)
            context?.fill(rect)
            let image = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()               
            return image!
        }
    }
    class Page: UIViewController {
        var v: UIImageView!
        override func viewDidLoad() {
            super.viewDidLoad()
            let image1 = UIImage.imageWithColor(UIColor.red)
            let image2 = UIImage.imageWithColor(UIColor.blue)
            v = UIImageView(image: image1,highlightedImage:image2)
            v!.frame = CGRect(x: 0,y: 0,width: 100,height: 100)
            self.view.addSubview(v!)
            let tapGestureRecognizer = UITapGestureRecognizer(target:self, action:#selector(imageTapped(_:)))
            v.isUserInteractionEnabled = true
            v.addGestureRecognizer(tapGestureRecognizer)
        }
        func imageTapped(_ img: AnyObject)
        {
            v.isHighlighted = !v!.isHighlighted
        }
    }

通过UITapGestureRecognizer手势识别类可以为UIImageView添加触控事件，此特性常常可以被利用来制作图片按钮。UIImageView本身支持正常图片和高光图片的切换，当轻按事件发生时，可以通过设置属性isHighlighted来切换图片的显示。




