

## 如何一拖tableview到底的时候更新数据

拖动tableView到最底下，随即更新之后的数据，是一个常见的应用场景。使用MJRefresh，或者GTMRefresh都是不错的更新方法。但是自己做的话，就可以更加简单，而无需第三方依赖，并且可以了解到第三方库的原理。

如下案例，显示了一个超出一屏幕的数据的tableview,当拖动到接近最下的时候（有一个值设为为100，就是还差100就到最底下），就会触发一次onRefresh函数的执行：

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
        var a : Table!
        override func viewDidLoad() {
            super.viewDidLoad()
            a  = Table()
            a.frame = UIScreen.main.bounds
            self.view.addSubview(a)
        }
     }
     
    class Table : UITableView,UITableViewDataSource,UITableViewDelegate{
        let lang = "java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,java,swift,js,".components(separatedBy: ",")
        let os = ["Windows","OS X","Linux"]
        override init(frame: CGRect, style: UITableViewStyle) {
            super.init(frame:frame,style:style)
            self.dataSource = self
            self.delegate = self
    //        lang = split(langStr) {$0 == ","}
        }
        required init?(coder aDecoder: NSCoder) {
            super.init(coder:aDecoder)
        }
        func numberOfSections(in: UITableView) -> Int {
            return 1
        }
        func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return lang.count
        }
        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let arr = lang
            let a = UITableViewCell(style: .default, reuseIdentifier: nil)
            a.textLabel?.text = String(describing:arr[indexPath.row])
            return a
        }
        let threshold : CGFloat = 100.0 // threshold from bottom of tableView
        var isLoadingMore = false // flag
        func onRefresh(){
            print("new data")
        }
        private func addObserver() {
            self.addObserver(self, forKeyPath: "contentOffset", options: .new, context: nil)
        }
        
        private func removeAbserver() {
            self.removeObserver(self, forKeyPath:"contentOffset")
        }
        var curentContentHeight : CGFloat = 0
        override open func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
            print(contentOffset.y)
        }
        func scrollViewDidScroll(_ scrollView: UIScrollView) {
    //        print(scrollView.contentOffset.y,self.contentSize.height,UIScreen.main.bounds.height)
            
            let contentOffset = scrollView.contentOffset.y
            let maximumOffset = scrollView.contentSize.height - scrollView.frame.size.height;
    //        print(maximumOffset ,contentOffset)
            // 还差100就翻到底的时候 。。。
            if !isLoadingMore && (maximumOffset - contentOffset <= threshold) {
                self.isLoadingMore = true
                let when = DispatchTime.now() + 2
                DispatchQueue.main.asyncAfter(deadline: when) {
                    self.onRefresh()
                    self.isLoadingMore = false
                }
            }
        }
     }
     
拖动tableview时，整体内容会发生偏移，以便增强操作效果。这里的实现的关键，是tableView做了UIScrollViewDelegate的委托，只要在加入

    func scrollViewDidScroll(_ scrollView: UIScrollView) 
实现，那么拖动时，就会调用此函数，让tableview类得到通知，在此函数内就可以判断加载新数据的时机。此处我们以一个变量threshold作为限定，当拖动到离最底下（也就是完整的内容的高度）还差这么多的时候，就开始触发onRefresh事件。此事件内，可以加入你需要加载数据的代码。


参考： http://anasimtiaz.com/?p=254
