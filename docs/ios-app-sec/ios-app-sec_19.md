## 第十五章：INDEX

### A

地址清理器（ASan），55

地址空间布局随机化（ASLR），8，53–54，87

`advertisingIdentifier`，235

`advertisingTrackingEnabled`，235

AES 算法，226–227

AFNetworking，122–124

`alloc`，19

*.app* 目录，78–79

苹果系统日志（ASL），161–164

应用程序结构，27–38

*Bundle* 目录，33–34

*Data* 目录，34–37

设备目录，32–33

*Documents* 目录，34–35

*Shared* 目录，37

`applicationDidEnterBackground`，20，167，179–180，183

应用扩展，140–144

`extensionPointIdentifier`，144

扩展点，140

`NSExtensionActivationRule`，142

`NSExtensionContext`，143

`NSExtensionItem`，143

`shouldAllowExtensionPointIdentifier`，143

第三方键盘，144

*Application Support* 目录，35

`applicationWillResignActive`，180

`applicationWillTerminate`，20，167，183

应用审查，3–4，10–12

躲避，11–12

应用商店，3–4

审查过程，10–12

绕过，11–12

ARC（自动引用计数），19

ASan（地址清理器），55

ASIHTTPRequest，122，124–125

ASL（苹果系统日志），161–164

ASLR（地址空间布局随机化），8，53–54，87

身份验证

生物识别，231–232

指纹认证的安全性，232

HTTP 基本身份验证，110–111，119–121

本地身份验证 API，231–232

TouchID，231–232

`LAContext`，231–232

自动修正，175–177

自动引用计数（ARC），19

`autoreleasepool`，19

### B

回溯（`bt`）命令，65–66

BEAST 攻击，117

生物识别，231–232

黑盒测试，77

黑名单，200

块，Objective-C

声明，18

向 JavaScript 暴露，150–151

蓝牙低功耗 (BTLE), 244

蓝牙个人区域网 (PAN), 125

Bonjour, 125

启动 ROM, 4

断点, 62

操作, 70–72, 164

条件, 72

启用/禁用, 64

设置, 62–64

暴力破解, PIN 码, 214

`bt` (回溯) 命令, 65–66

BTLE (蓝牙低功耗), 244

缓冲区溢出, 12, 193–196

示例, 194–195

防止, 195–196

*Bundle* 目录, 33–34

bundle ID, 33, 138

bundle seed ID, 218–219

BurpSuite, 43–47

### C

CA (证书授权机构), 114–115

CA 证书, 44

证书管理, 47

证书钉扎, 114–117, 124

打败, 96–97

缓存管理, 170–171

移除缓存数据, 171–174

*Caches* 目录, 35–36

缓存, 36

`CALayer`, 182–183

规范名称 (CN), 128–129

`canPerformAction`, 168–169

类别, Objective-C, 22–23

`CBPeripheralManager`, 246

CCCrypt, 186, 226

`CCHmac`, 229

CCRespring, 79

证书授权机构 (CA), 114–115

`certificateHandler`, 126

`CFArrayRef`, 112–113

`CFBundleURLSchemes`, 133

`CFDataRef`, 113

`CFPreferences`, 36, 178

`CFStream`, 48, 107, 128–129

`chflags` 命令, 42

clang, 51–53

`class-dump` 命令, 90, 92–93

`CLBeaconRegion`, 244, 246

`CLLocationManager`, 240, 244

CN (规范名称), 128–129

Cocoa, 14

Cocoa Touch, 14

代码段, 193–195

`codesign` 命令, 82

CommonCrypto, 151, 230, 230

CCCrypt, 226

`CompleteUnlessOpen`, 222–223

`CompleteUntilFirstUserAuthentication`, 220

`cookieAcceptPolicy`, 237

cookies, 36

接受策略, 237–238

被盗，114

*Cookies* 目录，36

禁用复制/粘贴，168–169

Cordova，150，154–157

*Cordova.plist*，156

`CoreBluetooth`，246

Core Data，204，223

CRIME 攻击，118

跨站脚本攻击（XSS），199–200

输入 sanitization，200–201

输出编码，201–202

`cryptid`，81，86–90

`cryptoff`，86–90

`cryptsize`，86–90

cURL，78

证书，93–94

Cycript，90，93–94

Cydia，31，77

Cydia Substrate，78，97–100

### D

DAC（自主访问控制），4–5

*Data* 目录，34–37

数据泄漏，161–188

Apple 系统日志，161–164

自动更正，175–177

断点操作，164

缓存管理，170–174

*dynamic-text.dat*，177

短暂会话，173

HTTP 缓存，169–174

本地存储，174

iCloud，161

避免，188

键盘记录器，177

`NSLog`，161–164

禁用，163

`NSURLSession`，173

粘贴板，164–169

`canPerformAction`，168

禁用复制/粘贴，168–169

*pasteboardDB* 文件，165–167

`pasteboardWithUniqueName`，165–167

保护数据，167–169

`UISearchBar`，165

擦除，167

快照，178–184

`applicationDidEnterBackground`，179–180

防止挂起，183–184

屏幕模糊，179–183

状态保存，184–187

`restorationIdentifier`，184–185

用户偏好设置，178

数据保护 API，7–8，219–225

类密钥，220

`CompleteUnlessOpen`，222–223

`CompleteUntilFirstUserAuthentication`，220

`DataProtectionClass`，223

数据保护授权，223–224

委托方法，224

检测，225

`FileProtectionComplete`，220–221

`isProtectedDataAvailable`，225

保护级别，220–223

`DataProtectionClass` 权限， 157

数据段， 193–195

`dataTaskWithRequest`， 18

数据盗窃， 161

`dd` 命令， 88

调试， 61–75

断点， 62

动作， 70–72

条件， 72

启用/禁用， 64

设置， 62–64

调试导航器， 65

`debugserver`， 81–84

连接到， 83

安装， 81–82

故障注入， 72–73

帧和变量， 64–68

lldb， 62–75

回溯（`bt`）命令， 65–66

`expr` 命令， 69

`frame select` 命令， 66–67

`frame variable` 命令， 66

`image list`， 87

`print object` 命令， 67–68

对象检查， 68

跟踪数据， 74

变量和属性， 69–70

调试导航器， 65

`debugserver`， 81–84

连接到， 83

安装， 81–82

`decodeRestorableStateWithCoder`， 184

解密二进制文件， 80–90

*.default_created.plist*， 32

*Default-Portrait.png*， 179

`defaults` 命令， 42

委托， 20

DES 算法， 226

反序列化， 21

开发者团队 ID， 138

设备目录， 32–33

设备密钥， 7

*device.plist*， 32

`didFinishNavigation`， 159–160

did 消息， 20

`didReceiveCertificate`， 126

反汇编， 使用 Hopper， 94–96

自主访问控制（DAC）， 4–5

*Documents* 目录， 34–35

不追踪， 236–237

`dpkg` 命令， 96， 99–101

DTrace， 55， 61

`dumpdecrypted` 命令， 80

`_dyld_get_image_name`， 10

dylibs， 10

动态分析， 55

动态打补丁， 11–12

### E

模拟器， *参见* 仿真器

`encodewithcoder`， 21–22

加密段， 84–90

加密， 211–230

AES， CCB 模式， 226–227

坏算法， 226

CommonCrypto， 225， 230

CCCrypt， 226

Curve25519， 222

数据保护 API，5，7–8，219–225

类密钥，220

`CompleteUnlessOpen`，222–223

`CompleteUntilFirstUserAuthentication`，220

`DataProtectionClass`，223

数据保护权限，223–224

委托方法，224

检测，225

`FileProtectionComplete`，220–221

`FileProtectionCompleteUnlessOpen`，222

`isProtectedDataAvailable`，225

保护级别，220–223

DES 算法，226

设备密钥，7

磁盘加密，5–7

椭圆曲线 Diffie-Hellman 算法，222

熵，227

文件密钥，7

完整磁盘加密，5–7

哈希，228–230

HMAC（哈希消息认证码），229–230

初始化向量（IV），226–227

钥匙串，6–7，113，186，211–219

API，7

备份，212

iCloud 同步，219

项目类，214

密钥层次结构，6–7

`kSecAttrAccessGroup`，218–219

保护属性，212–214

`SecItemAdd`，219

共享钥匙串，218–219

使用，214–217

包装器，217–218

密钥派生，227–228

密钥质量，227–228

锁箱，217

OpenSSL，228–229

RNCryptor，230

`SecRandomCopyBytes`，227

TLS（传输层安全性），127–129

权限，218，223

*entitlements.plist*，81–82

熵，227

Erica 工具，31，78

*/etc/hosts*，49

`EXC_BAD_ACCESS`，191

永久执行（XN），8–9

`expr`命令，69

`extensionPointIdentifier`，144

`extractIdentityAndTrust`，112–113

### F

故障注入，72–73

File Juicer，169，174

文件密钥，7

`FileProtectionComplete`，220–221

文件系统监控，58–59

Finder，42

指纹认证的安全性，232

法医攻击者，161

格式化字符串攻击，190–193

`NSString`, 192–193

防止, 191–193

Foundation 类, 14

帧和变量, 68

`frame select` 命令, 66–67

`frame variable` 命令, 66

全盘加密, 5–7

模糊测试, 55

### G

垃圾回收, 18

gdb, 62

地理定位, 238

精度, 239

`CLLocationManager`, 240

风险, 238–239

`get-task-allow`, 82

Google Toolbox for Mac, 202

GPS, 238

### H

`handleOpenURL`, 136

哈希, 228–230

哈希消息认证码 (HMAC), 229

`hasOnlySecureContent`, 159–160

HealthKit, 240–241

堆, 8, 53–54, 193

隐藏文件, 41–42

HMAC (哈希消息认证码), 229

Homebrew, 46, 88, 94, 99

钩子

使用 Cydia Substrate, 97–100

使用 Introspy, 100–103

Hopper, 94–96

HTML 实体, 201

编码, *见* 输出编码

HTTP 基本认证, 110–111, 119–121

HTTP 本地存储, 174

HTTP 重定向, 113–114

### I

iBeacons, 244–247

`CBPeripheralManager`, 246

`CLBeaconRegion`, 244–246

`CLLocationManager`, 244

`startMonitoringForRegion`, 244

iBoot, 4

iCloud, 35, 111, 161, 212, 219

避免, 187

IDA Pro, 94

`identifierForVendor`, 234

iExplorer, 28–29

iGoat, 178

`image list`, 87

实现，声明, 16–17

*Info.plist*, 33

`init`, 19

初始化向量 (IV), 226–227

`initWithCoder`, 21–22

`initWithContentsOfURL`, 206

注入攻击, 199–207

跨站脚本攻击 (XSS), 199–202

输入清理, 200–201

输出编码, 200–202

显示不可信数据, 202

谓词注入, 204–205

SQL 注入, 203–204

参数化 SQL, 203–204

SQLite，203–204

XML 注入，207

XML 外部实体，205–206

XPath，207

输入消毒，200–201

`installipa` 命令，80

InstaStock，12

Instruments，55–57

整数溢出，196–198

示例，197–198

防止，198

接口，声明，15–16

进程间通信，*参见* IPC（进程间通信）

Introspy，100–103

针对 iOS 的 Web 应用，147–160

IPA 安装控制台，78

*.ipa* 包，80

IPC（进程间通信），131–145

应用扩展，131，140

`extensionPointIdentifier`，144

扩展点，140

`isContentValid`，143

`NSExtensionActivationRule`，142

`NSExtensionContext`，143

`NSExtensionItem`，143

`shouldAllowExtensionPointIdentifier`，143

第三方键盘，143–144

`canOpenURL`，138

`handleOpenURL`，136

`isContentValid`，143

`openURL`，132–137

`sourceApplication`，136

`UIActivity`，139–140

`UIPasteboard`，144

通用链接，137–138

URL 方案，132–133

`CFBundleURLSchemes`，133

定义，132–133

劫持，136–137

验证 URL 和发送者，134

`iproxy` 命令，84

`isContentValid`，143

IV（初始化向量），226–227

ivars，15–17，91

### J

越狱检测，9–10

无用，9

越狱，4，9–10，77

JavaScript，11

在 Cordova 中执行，154–157

在`UIWebView`中执行，149–150

`stringByEvaluatingJavaScriptFromString`，149–150

JavaScript–Cocoa 桥接，150–157

JavaScriptCore，150–154

区块，150–151

`JSContext`，152–154

`JSExport`，151–152

Jekyll，12

即时（JIT）编译器，8–9，149

JRSwizzle，25

`JSContext`，152–154

`JSExport`，151–152

### K

`kCFStreamSSLLevel`，129

Keychain API, 6–7, 113, 186, 211

备份, 212

iCloud 同步, 219

`kSecAttrAccessGroup`, 218–219

保护属性, 212–214

`SecItemAdd`, 214–215, 219

`SecItemCopyMatching`, 216

`SecItemDelete`, 216

`SecItemUpdate`, 215

共享 Keychain, 218–219

用法, 214–217

包装器, 217–218

密钥派生, 227–228

键盘记录, 175–177

`killall` 命令, 79, 101

`kSecAttrAccessGroup`, 218–219

`kSecAttrAccessible`, 220

`kSecAttrSynchronizable`, 219

### L

`LAContext`, 231–232

`ldid` 命令, 97

LDID（链接标识编辑器）, 97

从 C 继承的遗留问题, 189–198

缓冲区溢出, 193–196

示例, 194–195

预防, 195–196

格式化字符串攻击, 190–193

`NSString`, 192–193

预防, 191–193

整数溢出, 196–198

示例, 197–198

预防, 198

`libc`, 8

*Library* 目录, 35–37

*Application Support* 目录, 35

*Caches* 目录, 35–36, 187

*Snapshots* 目录, 36

*Cookies* 目录, 36

*Preferences* 目录, 36

*Saved Application State* 目录, 37

`LIKE` 操作符, 205

链接标识编辑器 (LDID), 97

`lipo` 命令, 78, 85

lldb, 62–81, 83–84, 191

`backtrace`（`bt`）命令, 65–66

断点, 62

操作, 70–72, 164

条件, 72

启用/禁用, 64

设置, 62–64

`expr` 命令, 69

`frame select` 命令, 66–67

`frame variable` 命令, 66

`image list`, 87

`print object` 命令, 67–68

llvm, 90

本地身份验证 API, 231–232

Logos, 98

回环接口, 46–47

Lua, 12

### M

M7 处理器, 242

MAC（强制访问控制）, 4–5

MAC 地址， 234

Mach-O 二进制格式， 77， 85

MachOView， 88

MacPorts， 94

`malloc`， 197–198

强制访问控制（MAC）， 4–5

`MATCHES` 操作符， 205

`MCEncryptionNone`， 126

`MCEncryptionOptional`， 126

`MCEncryptionRequired`， 126

`MCSession`， 126

消息传递， 13–15

方法交换， 23–25

Mobile Safari， 44

MobileTerminal， 78

多点连接， 125–127

`certificateHandler`， 126

`didReceiveCertificate`， 126

加密， 125–127

### N

`netcat` 命令， 78

网络， 107–129

AFNetworking， 122–124

证书钉扎， 123–124

ASIHTTPRequest， 122， 124–125

`backgroundSessionConfiguration`， 117

`CFStream`， 48， 107， 128–129

`ephemeralSessionConfiguration`， 117

多点连接， 125–127

`certificateHandler`， 126

`didReceiveCertificate`， 126

加密， 125–127

`NSInputStream`， 49

`NSOutputStream`， 49

`NSStream`， 48， 107， 127–128

`NSURLSession`， 122

URL 加载系统， 107–122

HTTP 基本认证， 110–111， 119–121

HTTP 重定向， 113–114

`NSURLConnection`， 48， 108

`NSURLConnectionDelegate`， 109

`NSURLCredential`， 120

`NSURLCredentialStorage`， 110–111

`NSURLRequest`， 108

`NSURLResponse`， 108

`NSURLSession`， 48， 117

`NSURLSessionConfiguration`， 120–121

`NSURLSessionTaskDelegate`， 119

`sharedCredentialStorage`， 120–122

存储的 URL 凭证， 121–122

通知中心， 224–225

`NSCoder`， 185–187

`NSCoding`， 21–22

`NSData`， 113

`NSExtensionContext`， 143

`NSExtensionItem`， 143

`NSFileManager`， 221–223

`NSHTTPCookieStorage`， 237

`NSHTTPRequest`， 122

`NSInputStream`， 49

`NSLog`，95，161–164，192

禁用，163

`NSNotificationCenter`，224–225

`NSOperation`，122

`NSOutputStream`，49

`NSPredicate`，204–205

`NSStream`，48，107，127–128

`NSString`，192–193，195，202

`NSURAuthenticationMethodClientCertificate`，112

`NSURL`，188

`NSURLCache`，74–75，150

`NSURLConnection`，48，108，117

`NSURLConnectionDelegate`，109，114

`NSURLCredential`，113，120

`NSURLCredentialStorage`，110–111，121

`NSURLIsExcludedFromBackupKey`，35，187–188

`NSURLProtectionSpace`，109–111，122

`NSURLProtocol`，155

`NSURLRequest`，108，148–149

`NSURLResponse`，108

`NSURLSession`，48，117–122

`NSURLSessionConfiguration`，117–119

`NSURLSessionDataTask`，18

`NSURLSessionTaskDelegate`，119

`NSUserDefaults`，36，37，178

`NSUUID`，234

`NSXMLParser`，205–206

### O

Objective-C，13–25

blocks

声明，18

暴露给 JavaScript，150–151

类别，22–23

代码结构，15–17

令人愉快的，13

垃圾回收，18

实现，声明，16–17

ivars，15–16

消息传递，14–15

私有方法，缺失的部分，16

属性合成，17

引用计数，18–19

odcctools，78，84

OpenSSH，78

OpenSSL，94，228–229

`openssl` 命令，138

`openURL`，132–137

otool，53，78，84–86

检查二进制文件，90–92

输出编码，200–202

### P

p12 文件，113

参数化 SQL，203–204

*pasteboardDB* 文件，165–167

剪贴板，164–169

`canPerformAction`，168

禁用复制/粘贴，168–169

*pasteboardDB* 文件，165–167

`pasteboardWithUniqueName`，165–167

`UISearchBar`，165

`pasteboardWithUniqueName`，165–167

PhoneGap，11，150

物理攻击者，161

PIE（位置无关可执行文件），53–54

移除，87

plist 文件，29–31

转换，30–31

XML，29–30

`plutil` 命令，30–31

`popen`，10

位置无关可执行文件（PIE），53–54

移除，87

谓词注入，204–205

`LIKE` 运算符，205

`MATCHES` 运算符，205

通配符，204–205

谓词，205

`predicateWithFormat`，204–205

*Preferences* 目录，36

`printf` 命令，87，190–192

`print object` 命令，67–68

隐私问题，233–248

`advertisingTrackingEnabled`，235

蓝牙低功耗（BTLE），244

cookies，237–238

不追踪，236–237

地理定位，238–240

精度，239

`CLLocationManager`，240

`locationManager`，244

风险，238–239

GPS，238

HealthKit，240–241

iBeacons，244–247

`CBPeripheralManager`，246

`CLBeaconRegion`，244–246

`CLLocationManager`，244

`startMonitoringForRegion`，244

M7 处理器，242

MAC 地址，234

麦克风，233

隐私政策，247–248

接近跟踪，244–247

请求权限，243

唯一设备标识符（UDID），233–235

`advertisingIdentifier`，235

`identifierForVendor`，234

`NSUUID`，234

`uniqueIdentifier`，234

私有方法，16

属性合成，17

协议，20–22

声明，21–22

接近跟踪，244–247

代理设置，43–50

### Q

Quick Look，35，68

QuickType，177

### R

引用计数模型，18–19

`retain` 和 `release`，18–19

引用，强引用和弱引用，19

`release`, 18–19

远程设备擦除, 5, 6

`removeAllCachedResponses`, 75

重启, 79, 101

`restorationIdentifier`, 184–185

`retain`, 18–19

return-to-libc 攻击, 8

RNCryptor, 186, 230

`rootViewController`, 183

`rsync` 命令, 78

### S

安全字符串 API, 195

沙盒, 4–5

*已保存应用程序状态* 目录, 37

Seatbelt, 4–5

`SecCertificateRef`, 112–113

`SecIdentityRef`, 112–113

`SecItemAdd`, 186, 212, 215, 219

`SecItemCopyMatching`, 216

`SecItemDelete`, 216

`SecItemUpdate`, 215

`SecRandomCopyBytes`, 227

`SecTrustRef`, 112–113

安全启动, 4

SecureNSCoder, 186–187

`securityd`, 7

序列化, 21

`setAllowsAnyHTTPSCertificate`, 108

`setJavaScriptCanOpenWindowsAutomatically`, 159

`setJavaScriptEnabled`, 159–160

`setResourceValue`, 188

`setSecureTextEntry`, 175–177

`setShouldResolveExternalEntities`, 206

*共享* 目录, 37

`sharedCredentialStorage`, 120–122

`sharedHTTPCookieStorage`, 237

共享钥匙串, 218–219

`shouldAllowExtensionPointIdentifier`, 143

应该消息, 20

`shouldSaveApplicationState`, 20

`shouldStartLoadWithRequest`, 148

边加载, 77–80

有符号整数, 196

符号位, 51, 196

模拟器, 43–46

相机, 43

大小写敏感, 43

安装证书, 44

钥匙串, 43

PBKDF2, 43

代理, 44–46

信任库, 44

SpringBoard, 79

SQL 注入, 201, 203–204

参数化 SQL, 203–204

SQLite, 203–204

SSH, 28, 82

SSL, *见* TLS（传输层安全性）

SSL Conservatory, 115–117

SSL 终止开关, 96–97

堆栈, 8, 53–54, 190, 193

`startMonitoringForRegion`，244

状态保持，184–187

漏洞，184–185

`restorationIdentifier`，184–185

安全，185–187

静态分析，54

`std::string`，195

`strcat`，195

`strcpy`，194，195

`stringByEvaluatingJavaScriptFromString`，149–150

`strlcat`，195–196

`strlcpy`，195–196

强引用，19

stunnel，46

子类化，23

`synthesize`，17

`syslog`，162，190

### T

`task_for_pid-allow`，82

`tcpdump` 命令，78

tcpprox，49–50

TCP 代理，49–50

测试设备，42

文本段，85–86

Theos，97–98

精简二进制文件，85

`ThisDeviceOnly`，212

TLS（传输层安全性），108–119，127–129

BEAST 攻击，118

绕过验证，44–47，119

证书钉扎，114–117，123–124

CRIME 攻击，118

相互认证，112–113

`setAllowsAnyHTTPSCertificate`，108

验证，类别绕过，22

*tmp* 目录，37，80，187

今日屏幕，131

TOFU（首次使用即信任），127

TouchID，231–232

`LAContext`，231–232

传输层安全性，*见* TLS（传输层安全性）

Tribbles，51

首次使用即信任（TOFU），127

调整，Cydia Substrate，97

### U

UDID（唯一设备标识符），233–235

`advertisingIdentifier`，235

`identifierForVendor`，234

`NSUUID`，234

`uniqueIdentifier`，234

`UIActivity`，139–140

`UIAlertView`，183

UI 层，182–183

`UIPasteBoard`，144，164–169

`UIPasteboardNameFind`，165

`UIPasteboardNameGeneral`，165

`UIRequiredDeviceCapabilities`，34

`UIResponderStandardEditActions`，169

`UISearchBar`，165，175

`UITextField`，175

`UITextView`，175

`UIView`，182–183

`UIWebView`, 200, 201

`UIWindow`, 182–183

唯一设备标识符 (UDID), 233–235

`advertisingIdentifier`, 235

`identifierForVendor`, 234

`NSUUID`, 234

`uniqueIdentifier`, 234

`uniqueIdentifier`, 234

通用链接, 137–138

无符号整数, 196

URL 加载系统, 107

凭证持久性类型, 111

HTTP 基本认证, 110–111

HTTP 重定向, 113–114

`NSURLConnection`, 108

`NSURLConnectionDelegate`, 109

`NSURLCredential`, 120

`NSURLCredentialStorage`, 110–111

`NSURLRequest`, 108

`NSURLResponse`, 108

`NSURLSession`, 117–122

`NSURLSessionConfiguration`, 117–119

`NSURLSessionTaskDelegate`, 119

`sharedCredentialStorage`, 120–122

存储的 URL 凭证, 121–122

URL 协议, 132–133

`CFBundleURLSchemes`, 133

定义, 132–133

劫持, 136–137

验证 URL 和发送者, 134

USB，TCP 代理, 84

`usbmuxd` 命令, 84

用户偏好设置, 178

UUID, 27

`uuidgen`, 244

### V

Valgrind, 55

`vbindiff` 命令, 78, 88

`vfork`, 10

*.vimrc* 文件, 30

`vmaddr`, 88

### W

wardriving, 238

警告策略, 51

看门狗, 58–59

`watchmedo` 命令, 58–59

`weak_classdump`, 93

弱引用, 19

web 应用, 147–160

WebViews, 9, 147–160

Cordova, 154–157

风险, 156

`XmlHttpRequest`, 155

JavaScript, 149

在 Cordova 中执行, 154–157

在 `UIWebView` 中执行, 149–150

`stringByEvaluatingJavaScriptFromString`, 149–150

JavaScript–Cocoa 桥接, 150–157

JavaScriptCore, 149–154

blocks, 150–151

`JSContext`, 152–154

`JSExport`, 151–152

即时编译 (JIT) 编译器, 149

Nitro, 148, 149

`NSURLRequest`, 148–149

`UIWebView`, 147–150

WebKit, 11, 147–148

`WKWebView`, *参见* `WKWebView`

白名单, 149, 152, 200–201

will 消息, 20

`willSendRequestForAuthenticationChallenge`, 112

Wireshark, 46

`WKPreferences`, 160

`WKWebView`, 148, 158–160

`addUserScript`, 159

的好处, 159–160

`didFinishNavigation`, 159–160

`hasOnlySecureContent`, 159–160

`setJavaScriptCanOpenWindowsAutomatically`, 159

`setJavaScriptEnabled`, 159–160

`WKPreferences`, 160

`WKUserScript`, 159

`WKWebViewConfiguration`, 160

### X

`xcodebuild`, 190

Xcode 设置, 50–53, 55

警告, 51–53

Xcon, 10

XML 注入, 207

`NSXMLParser`, 205–206

XML 外部实体, 205–206

XPath, 207

XN（eXecute Never）, 8–9

XPath, 207

XSS（跨站脚本攻击）, 199–200

输入数据清理, 200–201

输出编码, 201–202

`xxd` 命令, 88

*iOS 应用安全*中使用的字体包括 New Baskerville、Futura、The Sans Mono Condensed 和 Dogma。该书采用 LATEX 2[*ε*] 包 `nostarch` 由 Boris Veytsman 进行排版（*(2008/06/06 v1.3 为 No Starch Press 排版书籍)*）。
