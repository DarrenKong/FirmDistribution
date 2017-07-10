# FirmDistribution
Enterprise package distribution.

# Usage

## 1.  IPA打包
1. 在工程中选择Product-Archive进入打包界面
2. 选择Export进入打包方式选择界面
3. 选择Save for Enterprise Deployment选项，Next
4. 选择对应的企业账号，然后继续即可
5. 接下来是对安装设备的要求选择，默认选择所有设备。第二个选项是指定特定类型设备方可安装。我们使用默认第一项，Next
6. 之后的界面是对应用的二次确认，确保APP配置准确无误。在窗口的左下方有一个Include manifest for over-the-air installation。该选项表示是否在生成.ipa文件的同时生成.plist文件，我们勾选上，Next
7. 接下来配置.plist文件，填写完之后，Export导出.ipa包和相应的.plist文件（建议将生成的.plist文件命名同APP名一致，方面后期管理）

## 2.  构建网站
1. 必备条件
-  需要购买一个苹果的企业版证书，价格$299/年。
-  网站需要支持HTTPS协议，用于访问下载.plist文件

2.  步骤
-  将.plist文件与.ipa文件上传至服务器供用户访问
- 创建一个包含如下代码的网页，用户轻点 Web 链接后会下载.plist文件，并触发下载和安装。
	```xml
	<a href="itms-services://?action=download-manifest&url=https://example.com/manifest.plist">Install App</a>
	```
- 配置服务器MIME类型。可能需要配置 Web 服务器，让.plist文件和.ipa文件可正确传输。

	**【警告】** 撤销分发证书会导致使用该证书签名的所有应用失效。只有万不得已时才应撤销证书，比如确定专用密钥已丢失或确信证书已遭破解。

3. 关于.plist文件
    > 	.plist文件是一个XML plist 文件，可供 Apple 设备用来从您的 Web 服务器上查找、下载和安装应用。清单文件由 Xcode 创建，使用的是您在共享用于企业分发的归档应用时所提供的信息。

    以下键是必填项：
    - URL：应用 (.ipa) 文件的完全限定 HTTPS URL
    - display-image：57 x 57 像素的 PNG 图像，在下载和安装过程中显示。指定图像的完全限定 URL
    - full-size-image：512 x 512 像素的 PNG 图像，表示 iTunes 中相应的应用
    - bundle-identifier：应用的包标识符，与 Xcode 项目中指定的完全一样
    - bundle-version：应用的包版本，在 Xcode 项目中指定
    - title：下载和安装过程中显示的应用的名称
    
    .plist文件还包含可选键。例如，如果应用文件太大，并且想要在执行错误检验（TCP 通信通常会执行该检验）的基础上确保下载的完整性，可以使用 MD5 键。通过指定项目数组的附加成员，您可以使用一个清单文件安装多个应用。
    
    示例 iOS 应用.plist文件
	```xml
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	    <dict>
	        <!-- array of downloads.-->
	        <key>items</key>
	        <array>
	            <dict>
	                <!-- an array of assets to download -->
	                <key>assets</key>
	                <array>
	                    <!-- software-package: the ipa to install.-->
	                    <dict>
	                        <!-- required. the asset kind.-->
	                        <key>kind</key>
	                        <string>software-package</string>
	                        <!-- optional. md5 every n bytes. will restart a chunk if md5 fails.-->
	                        <key>md5-size</key>
	                        <integer>10485760</integer>
	                        <!-- optional. array of md5 hashes for each "md5-size" sized chunk.-->
	                        <key>md5s</key>
	                        <array>
	                            <string>41fa64bb7a7cae5a46bfb45821ac8bba</string>
	                            <string>51fa64bb7a7cae5a46bfb45821ac8bba</string>
	                        </array>
	                        <!-- required. the URL of the file to download.-->
	                        <key>url</key>
	                        <string>https://www.example.com/apps/foo.ipa</string>
	                    </dict>
	                    <!-- display-image: the icon to display during download.-->
	                    <dict>
	                        <key>kind</key>
	                        <string>display-image</string>
	                        <!-- optional. indicates if icon needs shine effect applied.-->
	                        <key>needs-shine</key>
	                        <true/>
	                        <key>url</key>
	                        <string>https://www.example.com/image.57x57.png</string>
	                    </dict>
	                    <!-- full-size-image: the large 512x512 icon used by iTunes.-->
	                    <dict>
	                        <key>kind</key>
	                        <string>full-size-image</string>
	                        <!-- optional. one md5 hash for the entire file.-->
	                        <key>md5</key>
	                        <string>61fa64bb7a7cae5a46bfb45821ac8bba</string>
	                        <key>needs-shine</key>
	                        <true/>
	                        <key>url</key><string>https://www.example.com/image.512x512.jpg</string>
	                    </dict>
	                </array>
	                <key>metadata</key>
	                <dict>
	                    <!-- required -->
	                    <key>bundle-identifier</key>
	                    <string>com.example.fooapp</string>
	                    <!-- optional (software only) -->
	                    <key>bundle-version</key>
	                    <string>1.0</string>
	                    <!-- required. the download kind.-->
	                    <key>kind</key>
	                    <string>software</string>
	                    <!-- optional. displayed during download; typically company name -->
	                    <key>subtitle</key>
	                    <string>Apple</string>
	                    <!-- required. the title to display during the download.-->
	                    <key>title</key>
	                    <string>Example Corporate App</string>
	                </dict>
	            </dict>
	        </array>
	    </dict>
	</plist>
	```
苹果官方相关文档 [以无线方式安装企业内部应用](http://help.apple.com/deployment/ios/#/apda0e3426d7)
	
4. 关于正确配置.plist文件配置配置，供下载供供
    1.  在自己的服务器部署.plist时，直接提供服务器下载地址即可
    2.  将.plist文件部署在github上时，可以在进入.plist详细界面，点击RAW按钮获取正确的地址
    3.  另外阿里云、七牛也是一个不错的选择，我们可以将.plist文件上传到阿里云、七牛上面，需要配置ssl证书。

## 3.部署下载页面方式
1. 进入下载页面之后点击安装APP按钮之后才能下载安装
    ```xml
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <htmlxmlns="http://www.w3.org/1999/xhtml">
        <head>
            <metahttp-equiv="Content-Type"content="text/html; charset=utf-8"/>
            <title>应用名字</title>
        </head>
        <body>
            <h1style="font-size:80pt">如果点击无法下载安装，请复制超链接到浏览器中打开<h1/>
            <h1style="font-size:100pt">
            <a title="iPhone" href="itms-services://?action=download-manifest&url=https://example.com/manifest.plist">安装APP</a><h1/>
        </body>
    </html>
    ```
2. 进入下载页面之后自动下载安装
    ```xml
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <htmlxmlns="http://www.w3.org/1999/xhtml">
        <head>
            <metahttp-equiv="Content-Type"content="text/html; charset=utf-8"/>
            <title>应用名字</title>
            <script>
            var url ="https://example.com/manifest.plist";
            window.location ="itms-services://?action=download-manifest&url="+ url;
            </script>
        </head>
        <body>
        </body>
    </html>
    ```
3. 通过iOS应用内安装
    ```objc
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"itms-services://?action=download-manifest&url=https://example.com/manifest.plist"]];
    ```
4. 使用第三方托管平台（例如:[fir.im](https://fir.im/)、[蒲公英](https://www.pgyer.com/)、[TestFlight](https://developer.apple.com/testflight/)）