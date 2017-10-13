仓库github/DrawController(https://github.com/sascha/DrawerController)
是一个Swift语言编写的抽屉库。它可以传入一个中心视图控制器，一个左侧视图控制器，并由一个MaxDrawerWidth属性。创建完毕后，中心视图控制器的视图内容占据全部屏幕；当执行打开抽屉的函数时，整个中心控制的视图内容一起向右平移MaxDrawerWidth这么宽，并且显示左侧视图控制器的内容到屏幕上，内容占满屏幕，但是宽度不超过MaxDrawerWidth指定的值。

研究此代码库的时候，发现它是一个很好的解说Container View Controller[https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html] 概念的例子。这里面：

1. 类DrawerController就是一个Container View Controller
2. 其他的视图控制器就是Child View Controller
3. 第一个类和其他类构成了父子关系
4. 视图控制器内的视图其实都是放置到DrawerController.view之内的

我把此库内的关于动画和手势的代码全部去掉，专门留下特定于这些控制器交互的代码留下。从1000多行的代码抽取出来247行的代码如下：

     import UIKit
     var drawerController : DrawerPage?
     @UIApplicationMain
     class AppDelegate: UIResponder, UIApplicationDelegate {
        var window : UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            window = UIWindow()
            drawerController = DrawerPage()
            window!.rootViewController = drawerController
            window!.rootViewController!.view.backgroundColor = .blue
            window!.makeKeyAndVisible()
            return true
        }
     }
     class DrawerPage : DrawerBase{
        init(){
            super.init(CenterPage(),LeftPage())
        }
        // 哄编译器开心的代码
        required init?(coder aDecoder: NSCoder) {
            super.init(coder: aDecoder)
        }
     }
     class DrawerBase : DrawerController{
        init(_ center : UIViewController,_ left : UIViewController){
            super.init(centerViewController: center, leftDrawerViewController: left)
        }
        // 从入门到入门：
        // 1. What exactly is init coder aDecoder?
        // 2. What does the question mark means in public init?(coder aDecoder: NSCoder)?
        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
     }
     class LeftPage: UIViewController {
        var count = 0
        var label : UILabel!
        override func viewDidLoad() {
            super.viewDidLoad()
            self.view.backgroundColor = .white
            label   = UILabel()
            label.frame = CGRect(x: 100, y: 100, width: 120, height: 50)
            label.text =  "Left"
            view.addSubview(label)
            let button   = UIButton(type: .system)
            button.frame = CGRect(x: 120, y: 150, width: 120, height: 50)
            button.setTitle("Close",for: .normal)
            button.addTarget(self, action: #selector(buttonAction(_:)), for: .touchUpInside)
            view.addSubview(button)
        }
        @objc func buttonAction(_ sender:UIButton!){
            drawerController?.toggleLeftDrawerSide(animated: true, completion: nil)
        }
     }
     class CenterPage: UIViewController {
        var label : UILabel!
        override func viewDidLoad() {
            super.viewDidLoad()
            self.view.backgroundColor = .white
            label   = UILabel()
            label.frame = CGRect(x: 100, y: 100, width: 120, height: 50)
            label.text =  "Center"
            view.addSubview(label)
            let button   = UIButton(type: .system)
            button.frame = CGRect(x: 120, y: 150, width: 120, height: 50)
            button.backgroundColor = .blue
            button.setTitle("Left Page Drawer",for: .normal)
            button.addTarget(self, action: #selector(buttonAction(_:)), for: .touchUpInside)
            view.addSubview(button)
        }
        @objc func buttonAction(_ sender:UIButton!){
            drawerController?.toggleLeftDrawerSide(animated: true, completion: nil)
        }
        @objc func buttonAction1(_ sender:UIButton!){
    //        drawerController?.toggleRightDrawerSide(animated: true, completion: nil)
        }
     }
     // code imple
     open class DrawerController: UIViewController {
        fileprivate var _centerViewController: UIViewController?
        fileprivate var _leftDrawerViewController: UIViewController?
        fileprivate var _leftDrawerWidth = DrawerDefaultWidth
        open var centerViewController: UIViewController? {
            get {
                return self._centerViewController
            }
        }
        open var leftDrawerViewController: UIViewController? {
            get {
                return self._leftDrawerViewController
            }
        }
        open var leftDrawerWidth: CGFloat {
            get {
                return self._leftDrawerWidth
            }
        }
        open var shadowRadius = DrawerDefaultShadowRadius
        open var shadowOpacity = DrawerDefaultShadowOpacity
        fileprivate lazy var childControllerContainerView: UIView = {
            let a = UIView(frame: self.view.bounds)
            self.view.addSubview(a)
            return a
        }()
        fileprivate lazy var centerContainerView: DrawerCenterContainerView = {
            let centerFrame = self.childControllerContainerView.bounds
            let centerContainerView = DrawerCenterContainerView(frame: centerFrame)
            centerContainerView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
            centerContainerView.backgroundColor = UIColor.clear
            self.childControllerContainerView.addSubview(centerContainerView)
            return centerContainerView
        }()
        open fileprivate(set) var openSide: DrawerSide = .center {
            didSet {
                if self.openSide == .center {
                    self.leftDrawerViewController?.view.isHidden = true
                }
                self.setNeedsStatusBarAppearanceUpdate()
            }
        }
        public required init?(coder aDecoder: NSCoder) {
            super.init(coder: aDecoder)
        }
        override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
            super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        }
        public init(centerViewController: UIViewController, leftDrawerViewController: UIViewController?) {
            super.init(nibName: nil, bundle: nil)
            self.setCenter(centerViewController)
            self.setDrawer(leftDrawerViewController,for:.left)
        }
        fileprivate func childViewController(for drawerSide: DrawerSide) -> UIViewController? {
            var childViewController: UIViewController?
            switch drawerSide {
            case .left:
                childViewController = self.leftDrawerViewController
            case .center:
                childViewController = self.centerViewController
            }
            return childViewController
        }
        fileprivate func sideDrawerViewController(for drawerSide: DrawerSide) -> UIViewController? {
            var sideDrawerViewController: UIViewController?
            if drawerSide != .center {
                sideDrawerViewController = self.childViewController(for: drawerSide)
            }
            return sideDrawerViewController
        }
        fileprivate func updateShadowForCenterView() {
            self.centerContainerView.layer.masksToBounds = false
            self.centerContainerView.layer.shadowRadius = shadowRadius
            self.centerContainerView.layer.shadowOpacity = shadowOpacity
            if let shadowPath = centerContainerView.layer.shadowPath {
                let currentPath = shadowPath.boundingBoxOfPath
                if currentPath.equalTo(centerContainerView.bounds) == false {
                    centerContainerView.layer.shadowPath = UIBezierPath(rect: centerContainerView.bounds).cgPath
                }
            } else {
                self.centerContainerView.layer.shadowPath = UIBezierPath(rect: self.centerContainerView.bounds).cgPath
            }
        }
        open func toggleLeftDrawerSide(animated: Bool, completion: ((Bool) -> Void)?) {
            self.toggleDrawerSide(.left, animated: animated, completion: completion)
        }
        open func toggleDrawerSide(_ drawerSide: DrawerSide, animated: Bool, completion: ((Bool) -> Void)?) {
            assert({ () -> Bool in
                return drawerSide != .center
            }(), "drawerSide cannot be .None")
            if self.openSide == DrawerSide.center {
                self.openDrawerSide(drawerSide,completion: completion)
            } else {
                if (drawerSide == DrawerSide.left && self.openSide == DrawerSide.left) {
                    self.closeDrawer(animated: animated, completion: completion)
                } else if completion != nil {
                    completion!(false)
                }
            }
        }
        fileprivate func openDrawerSide(_ drawerSide: DrawerSide,completion: ((Bool) -> Void)?) {
            assert({ () -> Bool in
                return drawerSide != .center
            }(), "drawerSide cannot be .None")
            if let vc = self.leftDrawerViewController{
                vc.view.isHidden = false
                var newFrame: CGRect
                newFrame = view.frame
                newFrame.size.width = _leftDrawerWidth
                vc.view.frame = newFrame
                //            vc.view.frame = vc.evo_visibleDrawerFrame
            }
            func panRight(_ view : UIView,_ value : CGFloat){
                var newFrame: CGRect
                newFrame = view.frame
                newFrame.origin.x = value
                view.frame = newFrame
            }
            panRight(centerContainerView, _leftDrawerWidth)
            self.openSide = drawerSide
        }
        fileprivate func setDrawer(_ vc: UIViewController?, for drawerSide: DrawerSide) {
            assert({ () -> Bool in
                return drawerSide != .center
            }(), "drawerSide cannot be .None")
            let currentSideViewController = self.sideDrawerViewController(for: drawerSide)
            if currentSideViewController == vc {
                return
            }
            self._leftDrawerViewController = vc
            if vc != nil {
                self.addChildViewController(vc!)
                vc!.didMove(toParentViewController: self)
                self.childControllerContainerView.addSubview(vc!.view)
                self.childControllerContainerView.sendSubview(toBack: vc!.view)
                vc!.view.isHidden = true
            }
        }
        fileprivate func setCenter(_ vc: UIViewController?) {
            self._centerViewController = vc
            if vc != nil {
                self.addChildViewController(vc!)
                vc!.didMove(toParentViewController: self)
                self.centerContainerView.addSubview(vc!.view)
                self._centerViewController!.view.frame = self.centerContainerView.bounds
                self.updateShadowForCenterView()
            }
        }
        open func closeDrawer(animated: Bool, completion: ((Bool) -> Void)?) {
            let newFrame = self.childControllerContainerView.bounds
            self.setNeedsStatusBarAppearanceUpdate()
            self.centerContainerView.frame = newFrame
            self.openSide = .center
        }
        open override func viewDidLoad() {
            super.viewDidLoad()
            self.view.backgroundColor = UIColor.black
        }
        static let DrawerDefaultWidth: CGFloat = 210.0 //280.0
        static let DrawerDefaultShadowRadius: CGFloat = 10.0
        static let DrawerDefaultShadowOpacity: Float = 0.8
        typealias  DrawerCenterContainerView =  UIView
        public enum DrawerSide: Int {
            case center
            case left
        }
     }
     
这里的代码，很关键的是在DrawerController内的view中，创建了一个子视图叫做childControllerContainerView，其内在创建一个子视图centerContainerView，它们几个的大小都是屏幕大小，视图层次为：

    --view
    ----childControllerContainerView
    ------centerContainerView

并约定好:

1. 全部的Center View Controller的视图全部放置到centerContainerView内
2. 全部的left View Controller和right View Controller的视图全部放置到centerContainerView内
3. left View Controller和right View Controller的视图放置完毕后，需要首先隐藏起来，并且sendToBack，保证它是不可见的，也不会遮挡任何Center View Controller的内容

这样当指定打开抽屉函数时，centerContainerView整体平移，如果打开左边的抽屉，就向右移动，反之亦然。从而Center View Controller的视图也就跟着平移。并同时把抽屉视图（left View Controller和right View Controller的视图）显示出来，但是宽度不超过MaxDrawerWidth。

通过以上分析，可以加强对Container View Controller概念的理解。


