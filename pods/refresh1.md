
UIRefreshView怎么做到的？这里的案例解释了做法。

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
        let arr = ["swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby","swift","obj-c","ruby"]
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
            if refreshHeader != nil{
                if self.tableView.isDragging{
                    if -scrollView.contentOffset.y - scrollView.contentInset.top > (refreshHeader?.frame.height)! {
                        refreshHeader?.state = .pulling
                    }else{
                        refreshHeader?.state = .idle
                        refreshHeader?.orginY = scrollView.contentInset.top
                    }
                }else if refreshHeader?.state == .pulling{
                    refreshHeader?.state = .refreshing
                    InsetTop = (refreshHeader?.orginY)! + (refreshHeader?.frame.height)!
                    doRefresh()
                }
            }
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
                    self.delay(0.8){
                        self.footer?.state = .refreshing
                        scrollView.contentOffset.y = self.beginDragOffset + 25
                        self.delay(2){
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
        func delay(_ s : Double ,_ done :(()->Void)? ){
            let when = DispatchTime.now() + s
            DispatchQueue.main.asyncAfter(deadline: when) {
                done?()
            }
        }
        func doRefresh(){
            onRefresh(){
                self.refreshHeader?.state = .idle
            }
        }
        func doLoadMore(){
            onLoadMore(){
                self.footer?.state = .idle
            }
        }
        func onRefresh(_ done : (()->Void)?){
            if let done = done {
                let when = DispatchTime.now() + 3
                DispatchQueue.main.asyncAfter(deadline: when) {
                    done()
                }
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
        var refreshHeader : RefreshHeader?
        var footer : RefreshFooter?
        override func viewDidLoad() {
            super.viewDidLoad()
            tableView.separatorStyle = .none
            addHeader()
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
        

        func addHeader(){
            refreshHeader = RefreshHeader()
            refreshHeader?.scrollView = self.tableView
            self.tableView.insertSubview(refreshHeader!, at: 0)
        }
        func addFooter(){
            let footerOffset = tableView.contentSize.height > tableView.frame.height ? tableView.contentSize.height: tableView.frame.height
            footer = RefreshFooter(Int(footerOffset - nav!.navigationBar.frame.height ))
            self.tableView.insertSubview(footer! , at: 0)
        }
    }
