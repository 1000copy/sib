
## TableView内置WebView的高度计算方法

Cell高度自适应的问题真多。现在，如果内部有webView，内容动态装入，大小也是各不相同的，并且高度必须根据内容，而不是view本身的高度来适应，怎么办呢？特别是如果有多个webView的情况下。
这样就可以了：

```swift
  import UIKit
  @UIApplicationMain
  class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.makeKeyAndVisible()
        window?.rootViewController = TableViewController()
        return true
    }
  }
  class Cell:UITableViewCell{
    var webView = UIWebView()
    override func layoutSubviews() {
        self.contentView.addSubview(webView)
    }
  }
  class TableViewController: UITableViewController, UIWebViewDelegate
  {
    fileprivate let reuserId = "BookTableViewCellIdentifier"
    override func viewDidLoad() {
        self.tableView.register(Cell.self, forCellReuseIdentifier: reuserId)
    }
    var content : [String] = ["test1<br>test1<br>test1<br>test1<br>", "test22<br>test22<br>test22<br>test22<br>test22<br>test22"]
    var contentHeights : [CGFloat] = [0.0, 0.0]
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return content.count
    }
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: reuserId, for: indexPath) as! Cell
        let htmlString = content[indexPath.row]
        var htmlHeight = contentHeights[indexPath.row]
        if htmlHeight == 0.0{
            htmlHeight = 100
        }
        cell.webView.tag = indexPath.row
        cell.webView.delegate = self
        cell.webView.loadHTMLString(htmlString, baseURL: nil)
        cell.webView.frame = CGRect(x:10, y:0, width:cell.frame.size.width, height:htmlHeight)
        return cell
    }
    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat{
        return contentHeights[indexPath.row]
    }
    func webViewDidFinishLoad(_ webView: UIWebView)
    {
        if (contentHeights[webView.tag] != 0.0)
        {
            // we already know height, no need to reload cell
            return
        }
        contentHeights[webView.tag] = webView.scrollView.contentSize.height
        print(contentHeights)
        tableView.reloadRows(at: [IndexPath(row: webView.tag, section:0)], with: .automatic)
    }
  }
```

首先webview只有装入内容完成，才知道内容的高度的，因此需要通过委托，监听事件webViewDidFinishLoad，在此处获得高度。

其次，多个webview会需要在一个事件内监听，因此，必须区别它们，代码中采用了webview.tag属性，赋值为此webview的cell的indexPath，把它记录到高度数组内。如果数组内的高度有值了，那么就不必在写入了。

最后，tableView是可以局部刷新的，那个高度变了就重新装入那个cell，做法就是使用reloadRows方法即可。

此案例的好处是简单，可以用来说明问题，但是明显存在着适应性问题，比如webview.tag内存储的是indexPath.row，而不是indexPath，那么在有多个section的TableView时，此代码就无法完成功能了。接下来的代码会更有通用性：

     import UIKit
     @UIApplicationMain
     class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            window = UIWindow(frame: UIScreen.main.bounds)
            window?.makeKeyAndVisible()
            window?.rootViewController = TableViewController()
            return true
        }
     }
     class Cell:UITableViewCell{
        var title  = UILabel()
        var webView = TJWeb()
        override func layoutSubviews() {
            self.contentView.addSubview(webView)
            self.contentView.addSubview(title)
            // layout code
            let view2 = webView
            let view1 = title
            let view = self.contentView
            view2.translatesAutoresizingMaskIntoConstraints = false
            view1.translatesAutoresizingMaskIntoConstraints = false
            let views = ["view1":view1,"view2":view2]
            let hConstraint1=NSLayoutConstraint.constraints(withVisualFormat: "H:|-[view1]-|", options: NSLayoutFormatOptions(rawValue: 0), metrics: nil, views: views)
            view.addConstraints(hConstraint1)
            let hConstraint2=NSLayoutConstraint.constraints(withVisualFormat: "H:|-[view2]-|", options: NSLayoutFormatOptions(rawValue: 0), metrics: nil, views: views)
            view.addConstraints(hConstraint2)
            let vConstraint1=NSLayoutConstraint.constraints(withVisualFormat: "V:|-5-[view1]-5-[view2]-|", options: NSLayoutFormatOptions(rawValue: 0), metrics: nil, views: views)
            view.addConstraints(vConstraint1)
        }
     }
     class TJWeb : UIWebView,UIWebViewDelegate{
        var indexPath : IndexPath!
        override func layoutSubviews() {
            isUserInteractionEnabled = false
        }
     }
     class Item {
        init(_ title : String?,_ content : String?){
            self.title = title
            self.content = content
        }
        var title : String?
        var content : String?
     }
     var data = [
            Item("1","test1<br>test1<br>test1<br>test1<br>"),
            Item("2","test22<br>test22<br>test22<br>test22<br>test22<br>test22"),
            Item("3","test22<br>test22<br>test22<br>test22<br>test22<br>test22"),
            Item("4","test22<br>test22<br>test22<br>test22<br>test22<br>test22"),
            Item("5","test22<br>test22<br>test22<br>test22<br>test22<br>test22"),
            Item("6","test22<br>test22<br>test22<br>test22<br>test22<br>test22"),
     ]
     class TableViewController: UITableViewController, UIWebViewDelegate
     {
        fileprivate let reuserId = "BookTableViewCellIdentifier"
        override func viewDidLoad() {
            self.tableView.register(Cell.self, forCellReuseIdentifier: reuserId)
        }
        var contentHeights : [IndexPath:CGFloat] = [:]
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return data.count
        }
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: reuserId, for: indexPath) as! Cell
            let htmlString = data[indexPath.row].content
            var htmlHeight = contentHeights[indexPath]
            if htmlHeight == nil{
                htmlHeight = 100
            }
            cell.webView.indexPath = indexPath
            cell.webView.delegate = self
            cell.webView.loadHTMLString(htmlString!, baseURL: nil)
            cell.title.text = data[indexPath.row].title
            cell.title.backgroundColor = .blue
            // layout section
    //        cell.webView.frame = CGRect(x:0.0, y:webTop, width:cell.frame.size.width, height:htmlHeight!)
    //        cell.title.frame   = CGRect(x:0.0, y: 5.0 , width:cell.frame.size.width, height:titleHeight)
            return cell
        }
        let fixWidth : CGFloat = 30
        override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat{
            if contentHeights[indexPath] != nil {
                return contentHeights[indexPath]! + fixWidth
            }
            return 44
        }
        func webViewDidFinishLoad(_ webView: UIWebView)
        {
            let web = webView as! TJWeb
            if (contentHeights[web.indexPath] != nil)
            {
                // we already know height, no need to reload cell
                return
            }
            contentHeights[web.indexPath] = webView.scrollView.contentSize.height
            print(contentHeights)
            tableView.reloadRows(at: [web.indexPath], with: .automatic)
        }
     }

要点在于，传入cell的webview是子类化的了，新的类内有一个属性，可以用来存在此webview所在的cell的indexPath。

还有就是，webview高度一旦确定，就会重新装入对应的cell，并刷新此cell的高度，而webview的高度，使用了布局技术，让它可以跟随cell高度而伴随增大。这样就不必自己计算视图的frame了。

