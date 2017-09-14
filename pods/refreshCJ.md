代码都在此仓库内：

    https://github.com/1000copy/cjrefresh
不但有下拉刷新，还有上拉刷新。不过上拉刷新写的还不太理想。需要进一步优化。

以下为正文。

当向下拉动UITableView时，如果使用了UIRefreshControl，那么会按着操作次序，出现几个有趣的UI变化：

1. 开始拖动时，可以看到tableview上方出现一个横条，宽度为屏幕，文字为idle。tableview内容随着拖动向下移动
2. 继续拖放，超过一定范围，开始显示为pulling
3. 此时如果松开拖动，此横条就会固定在tableview上方并显示为refreshing
4. 完成用户数据刷新后，此横条向上消失，tableview的内容恢复到拖动前的本位上


UIRefreshView怎么做到的？这里的案例解释了做法。

    var nav :  UIViewController?
    import UIKit
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            self.window = UIWindow(frame: UIScreen.main.bounds)
            nav = Nav()
    //        nav = LangTableViewController(style:.plain)
            nav?.view.backgroundColor = .blue
            self.window!.rootViewController = nav
            self.window?.makeKeyAndVisible()
            return true
        }
    }
    class Nav: UINavigationController {
        var count = 0
        var label : UILabel!
        override func viewDidLoad() {
            super.viewDidLoad()
            self.view.backgroundColor = .white
            self.pushViewController(LangTableViewController(style:.plain), animated: true)
            print(self.navigationBar.bounds)
        }
    }
    class LangTableViewController : RefreshableTVC{
        let arr = ["swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby"]
        //let arr = ["swift","obj-c","ruby","swift"]
        let MyIdentifier = "cell"
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return arr.count
        }
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let a = tableView.dequeueReusableCell(withIdentifier: MyIdentifier)
            a!.textLabel?.text = arr[indexPath.row]
            return a!
        }
        override func viewDidLoad() {
            super.viewDidLoad()
            tableView.register(UITableViewCell.self, forCellReuseIdentifier: MyIdentifier)
        }
    }
    // implements
    enum RefreshState {
        case idle
        case pulling
        case refreshing
    }
    class RefreshHeader : UIView{
        var label : UILabel?
        init(){
            let RefreshHeaderHeight  = 30
            let frame = CGRect(x: 0, y: -RefreshHeaderHeight, width: Int(UIScreen.main.bounds.width), height: RefreshHeaderHeight)
            super.init(frame: frame)
            state = .idle
            label = UILabel()
            label?.frame = CGRect(x: 0, y: 0, width: 100, height: 20)
            addSubview(label!)
            backgroundColor = UIColor.cyan
        }
        var text: String?{
            didSet{
                label?.text = text
                makeCenter(label!, self)
            }
        }
        func makeCenter(_ view : UIView, _ parentView : UIView){
            let x = (parentView.frame.width / 2) - (view.frame.width / 2)
            let y = (parentView.frame.height / 2) - (view.frame.height / 2)
            let rect = CGRect(x: x, y: y, width: view.frame.width, height: view.frame.height)
            view.frame = rect
        }
        var tableView : UITableView!
        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
        var marginTop : CGFloat?
        var state : RefreshState? {
            didSet{
                if state == .idle {
                    text = "idle"
                    if marginTop != nil {
                        tableView.contentInset.top = marginTop!
                    }else{
                        marginTop = tableView.contentInset.top
                    }
                }else if state == .pulling {
                    text = "pulling"
                }else if state == .refreshing {
                    text = "refreshing"
                    tableView.contentInset.top +=  (frame.height)
                }
            }
        }
    }
    class RefreshableTVC : UITableViewController{
        override init(style: UITableViewStyle) {
            super.init(style:style)
        }
        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
        // refreshHeader 完整的漏出来了
        func isExposed()->Bool{
            print( tableView.contentOffset.y ,tableView.contentInset.top , refreshHeader.frame.height)
            return -tableView.contentOffset.y - tableView.contentInset.top > refreshHeader.frame.height
        }
        override func scrollViewDidScroll(_ scrollView: UIScrollView) {
            if refreshHeader != nil{
                if refreshHeader?.state == .refreshing{
                    return
                }
                if self.tableView.isDragging{
                    if isExposed() {
                        refreshHeader?.state = .pulling
                    }else{
                        refreshHeader?.state = .idle
                    }
                }else if refreshHeader?.state == .pulling{
                    refreshHeader?.state = .refreshing
                    doRefresh()
                }
            }
        }
        func doRefresh(){
            onRefresh(){
                self.refreshHeader?.state = .idle
            }
        }
        func onRefresh(_ done : (()->Void)?){
            if let done = done {
               delay(3){
                    done()
                }
            }
        }
        var refreshHeader : RefreshHeader!
        override func viewDidLoad() {
            super.viewDidLoad()
            tableView.separatorStyle = .none
            addHeader()
        }
        func addHeader(){
            refreshHeader = RefreshHeader()
            refreshHeader.tableView = self.tableView
            self.tableView.addSubview(refreshHeader)
        }
    }
    func delay(_ s : Double ,_ done :(()->Void)? ){
        let when = DispatchTime.now() + s
        DispatchQueue.main.asyncAfter(deadline: when) {
            done?()
        }
    }
    
当调用override func scrollViewDidScroll(_ scrollView: UIScrollView) ，可以根据当前是否为拖动状态(isDragging),以及refreshHeader是否完整的漏出来作为依据，设置refreshHeader的状态为idle,pulling,refreshing，如果状态已经为refreshing，那么在此状态下，再次发起个拖动，就可以忽略它对refreshHeader状态的影响。

特别注意，数据装入完毕，界面稳定后，数值tableView.contentOffset.y，数值tableView.contentInset.top并不是0。因此此场景下，ViewController外还有一个NavigationController因此，要显示tableview内容，这两个值都是存在的。在我的环境下（iPhoneSE Simulator），两个值分为为-64,64。

原原本本的和需求对照下：

1. 开始拖动时，出现一个横条，是通过addHeader把refreshHeader实例化，并加入到tableview作为子视图。子视图位置为负数的自身高度。因此，在拖动前，是看不到此视图的。
2. 继续拖放，视图就会逐步显露，超过一定范围，状态切换到pulling
3. 此时如果松开拖动，默认情况下，tableview会全部移动回到原位，refreshHeader也会完全隐藏。但是因为此时设置了tableView.contentInset.top为初始值加上refreshHeader的高度，因此在refreshing状态下， 此横条就会固定在tableview上方
4. 完成用户数据刷新后，设置了tableView.contentInset.top为初始值。此横条向上消失，tableview的内容恢复到拖动前的本位上

感觉自己好啰嗦。

下拉刷新的示例源码在此：

    var nav :  UINavigationController?
    import UIKit
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            self.window = UIWindow(frame: UIScreen.main.bounds)
            nav = Nav()
            nav?.view.backgroundColor = .blue
            self.window!.rootViewController = nav
            self.window?.makeKeyAndVisible()
            return true
        }
    }
    class Nav: UINavigationController {
        var count = 0
        var label : UILabel!
        override func viewDidLoad() {
            super.viewDidLoad()
            self.view.backgroundColor = .white
            self.pushViewController(LangTableViewController(style:.plain), animated: true)
            print(self.navigationBar.bounds)
        }
    }
    class LangTableViewController : RefreshableTVC{
        let arr = ["swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby"]
        //    let arr = ["swift","obj-c","ruby","swift"]
        let MyIdentifier = "cell"
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return arr.count
        }
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let a = tableView.dequeueReusableCell(withIdentifier: MyIdentifier)
            a!.textLabel?.text = arr[indexPath.row]
            return a!
        }
        override func viewDidLoad() {
            super.viewDidLoad()
            tableView.register(UITableViewCell.self, forCellReuseIdentifier: MyIdentifier)
        }
    }
    // implements
    enum RefreshState {
        case idle
        case pulling
        case refreshing
    }
    class RefreshFooter : UIView{
        var label : UILabel?
        init(_ y : Int){
            let RefreshHeaderHeight  = 30
            let frame = CGRect(x: 0, y: y, width: Int(UIScreen.main.bounds.width), height: RefreshHeaderHeight)
            super.init(frame: frame)
            state = .idle
            label = UILabel()
            label?.frame = CGRect(x: 0, y: 0, width: 100, height: 20)
            addSubview(label!)
            backgroundColor = UIColor.cyan
        }
        var text: String?{
            didSet{
                label?.text = text
                makeCenter(label!, self)
            }
        }
        func makeCenter(_ view : UIView, _ parentView : UIView){
            let x = (parentView.frame.width / 2) - (view.frame.width / 2)
            let y = (parentView.frame.height / 2) - (view.frame.height / 2)
            let rect = CGRect(x: x, y: y, width: view.frame.width, height: view.frame.height)
            view.frame = rect
        }
        var scrollView : UIScrollView?
        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
        var orginY : CGFloat?
        var state : RefreshState? {
            didSet{
                if state == .idle {
                    text = "idle"
                    if let o = orginY{
                        scrollView?.contentInset.top = o
                    }
                }else if state == .pulling {
                    text = "pulling"
                }else if state == .refreshing {
                    text = "refreshing"
                }
            }
        }
    }
    class RefreshableTVC : UITableViewController{
        override init(style: UITableViewStyle) {
            super.init(style:style)
        }
        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
        var InsetTop : CGFloat{
            get {
                return tableView.contentInset.top
            }
            set{
                tableView.contentInset.top = newValue
            }
        }
        private var lastContentOffset: CGFloat = 0
        private var isMoveUp: Bool = false
        private var beginDragOffset: CGFloat = 0
        override func scrollViewDidScroll(_ scrollView: UIScrollView) {
            if footer != nil{
                if self.tableView.isDragging{
                    let footerVisibleSection  = -(curentContentHeight - UIScreen.main.bounds.height - scrollView.contentOffset.y)
                    if beginDragOffset == 0{
                        beginDragOffset = curentContentHeight - UIScreen.main.bounds.height
                        print("beginDragOffset:",self.beginDragOffset )
                    }
                    if footerVisibleSection > 20 {
                        footer?.state = .pulling
                    }else if footerVisibleSection <= 20 {
                        footer?.state = .idle
                    }
                }else if footer?.state == .pulling {
                    delay(0.8){
                        self.footer?.state = .refreshing
                        scrollView.contentOffset.y = self.beginDragOffset + 25
                        delay(2){
                            if self.beginDragOffset != 0 {
                                scrollView.contentOffset.y  = self.beginDragOffset
                                self.footer?.state = .idle
                            }
                            self.beginDragOffset = 0
                            self.doLoadMore()
                        }
                    }
                }
            }
        }
        func doLoadMore(){
            onLoadMore(){
                self.footer?.state = .idle
            }
        }
        func onLoadMore(_ done : (()->Void)?){
            if let done = done {
                let when = DispatchTime.now() + 3
                DispatchQueue.main.asyncAfter(deadline: when) {
                    done()
                }
            }
        }
        var footer : RefreshFooter?
        override func viewDidLoad() {
            super.viewDidLoad()
            tableView.separatorStyle = .none
            //        addHeader()
            addFooter()
            addObserver()
        }
        private func addObserver() {
            tableView?.addObserver(self, forKeyPath: "contentSize", options: .new, context: nil)
        }
        private func removeAbserver() {
            tableView?.removeObserver(self, forKeyPath:"contentSize")
        }
        var curentContentHeight : CGFloat = 0
        override open func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
            guard tableView.isUserInteractionEnabled else {
                return
            }
            print(tableView.contentSize.height)
            curentContentHeight = tableView.contentSize.height
            footer!.frame.origin.y = curentContentHeight
        }
        func addFooter(){
            let footerOffset = tableView.contentSize.height > tableView.frame.height ? tableView.contentSize.height: tableView.frame.height
            footer = RefreshFooter(Int(footerOffset - nav!.navigationBar.frame.height ))
            self.tableView.insertSubview(footer! , at: 0)
        }
    }
    func delay(_ s : Double ,_ done :(()->Void)? ){
        let when = DispatchTime.now() + s
        DispatchQueue.main.asyncAfter(deadline: when) {
            done?()
        }
    }
    
要点在于：

1. footer插入位置。如果tableview内容不满屏，那么插入到屏幕高度的位置，这样不拖动的时候，看不到footer，一旦上拉，立刻可以看到。方法是首先设置footer到屏幕高度位置，然后监视contentSize，如果变化并且大于屏幕高度，那么就设置contentSize.height为footer的新Y位置。
2. 切换状态也是根据contentOffset.y 来处理的，这和下拉刷新类似
3. 处理refreshing状态时，需要计算contentOffset.y的值，使得此数值代表的内容偏移，刚刚好可以完整显示footer。并且特别要注意，停止拖放后框架默认会把tableview内容偏移回复到拖放前的值，因此一定需要等到此恢复完全完成后在重新设置contentOffset，这就是为什么需要delay(0.8)的原因。也是因此，我觉得当前的实现并不优美，用户看起来也不舒服。估计也应该改成利用inset.bottom才好。
