可以使用AVFoundation框架来启动相机扫描二维码，把一个二维码转换为一个字符串。

如下应用，进入首页看到一个按钮和一个标签。点按钮的话，会触发一次扫描，把扫描到的二维码转换为字符串后，会显示在标签内。代码如下：
     
	import UIKit
    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {
        var window: UIWindow?
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
            self.window = UIWindow(frame: UIScreen.main.bounds)
            let page = Page1()
            self.window!.rootViewController = page
            self.window?.makeKeyAndVisible()
            return true
        }
    }
    var qrcode = ""
    class Page1: UIViewController {
        var count = 0
        var label : UILabel!
        var button1 = UIButton()
        var label1 = UILabel()
        override func viewDidLoad() {
            super.viewDidLoad()
            button1.frame = CGRect(x: 10, y: 130, width: 300, height: 20)
            button1.setTitle("Scan QR code",for: .normal)
            button1.addTarget(self, action: #selector(Success(_:)), for: .touchUpInside)
            view.addSubview(button1)
            label1.frame = CGRect(x: 10, y: 100, width: 300, height: 20)
            label1.text = "QRCode here"
            label1.backgroundColor = .red
            view.addSubview(label1)
        }
        func Success(_ sender: AnyObject) {
            label1.text = "QRCode here:\(qrcode)"
            self.present(ScannerViewController(), animated: true)
        }
    }
    // for Scan QR code test
    import AVFoundation
    import UIKit
    class ScannerViewController: UIViewController, AVCaptureMetadataOutputObjectsDelegate {
        var captureSession: AVCaptureSession!
        var previewLayer: AVCaptureVideoPreviewLayer!
        override func viewDidLoad() {
            super.viewDidLoad()
            view.backgroundColor = UIColor.blue
            captureSession = AVCaptureSession()
            let videoCaptureDevice = AVCaptureDevice.defaultDevice(withMediaType: AVMediaTypeVideo)
            let videoInput: AVCaptureDeviceInput
            do {
                videoInput = try AVCaptureDeviceInput(device: videoCaptureDevice)
            } catch {
                return
            }
            if (captureSession.canAddInput(videoInput)) {
                captureSession.addInput(videoInput)
            } else {
                failed();
                return;
            }
            let metadataOutput = AVCaptureMetadataOutput()
            if (captureSession.canAddOutput(metadataOutput)) {
                captureSession.addOutput(metadataOutput)
                metadataOutput.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
                metadataOutput.metadataObjectTypes = [AVMetadataObjectTypeQRCode]
            } else {
                failed()
                return
            }
            previewLayer = AVCaptureVideoPreviewLayer(session: captureSession);
            previewLayer.frame = view.layer.bounds;
            previewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
            view.layer.addSublayer(previewLayer);
            captureSession.startRunning();
        }
        func failed() {
            let ac = UIAlertController(title: "Scanning not supported", message: "Your device does not support scanning a code from an item. Please use a device with a camera.", preferredStyle: .alert)
            ac.addAction(UIAlertAction(title: "OK", style: .default))
            present(ac, animated: true)
            captureSession = nil
        }
        override func viewWillAppear(_ animated: Bool) {
            super.viewWillAppear(animated)
            if (captureSession?.isRunning == false) {
                captureSession.startRunning();
            }
        }
        override func viewWillDisappear(_ animated: Bool) {
            super.viewWillDisappear(animated)
            if (captureSession?.isRunning == true) {
                captureSession.stopRunning();
            }
        }
        func captureOutput(_ captureOutput: AVCaptureOutput!, didOutputMetadataObjects metadataObjects: [Any]!, from connection: AVCaptureConnection!) {
            captureSession.stopRunning()
            if let metadataObject = metadataObjects.first {
                let readableObject = metadataObject as! AVMetadataMachineReadableCodeObject;
                AudioServicesPlaySystemSound(SystemSoundID(kSystemSoundID_Vibrate))
                found(code: readableObject.stringValue);
            }
            dismiss(animated: true)
        }
        func found(code: String) {
            qrcode = code
            self.dismiss(animated: true, completion: nil)
        }
        override var prefersStatusBarHidden: Bool {
            return true
        }
        override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
            return .portrait
        }
    }

需要注意的是，首先必须在info.plist内指定NSCameraUsageDescription，如下：


 	<key>NSCameraUsageDescription</key>
	<string>scan QR</string>

否则会报错：

     [access] This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSCameraUsageDescription key with a string value explaining to the user how the app uses this data