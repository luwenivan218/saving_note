> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7312818383849799717)

> 译文地址：[codewithandrea.com/articles/fl…](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F "https://codewithandrea.com/articles/flutter-deep-links/")

深层链接正在改变移动应用程序的格局，而 Flutter 处于最前沿。想象一下：一位朋友向您发送了一个产品链接。你点击它，瞧！您不仅在应用程序中，而且正在查看确切的产品。魔法？不，这是深度链接！

在 Flutter 中，深层链接充当特定应用程序内容或功能的直接路径，这对于具有可共享内容的应用程序至关重要。当用户共享链接时，它们可以提供无缝的应用内体验，支持从营销活动到内容快捷方式的一切。

本指南将揭开 Flutter 中深度链接的神秘面纱。它涵盖了从原生 Android 和 iOS 设置到使用 GoRouter 处理导航的所有内容。

深层链接不仅使导航变得轻松，还可以让用户回访、提高参与度并简化内容共享。无论您是要构建第一个深度链接还是对现有内容进行微调，本指南都是深度链接成功的最终路线图。

[深层链接剖析](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23anatomy-of-a-deep-link "https://codewithandrea.com/articles/flutter-deep-links/#anatomy-of-a-deep-link")
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**深层链接是将用户发送到应用程序中的目的地**（而不是网页）的链接。要了解深层链接的组成部分，请考虑以下示例：

*   `https://codewithandrea.com/articles/parse-json-dart/`

深层链接包含一串称为 [URI](https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FUniform_Resource_Identifier "https://en.wikipedia.org/wiki/Uniform_Resource_Identifier") 的字符，其中包含：

*   **方案** - 这是链接的第一部分，通常显示为 http 或 https。该方案告诉我们在线获取资源时使用什么协议。
*   **主机** - 这是链接的域名。在我们的例子中，[codewithandrea.com](https://link.juejin.cn?target=http%3A%2F%2Fcodewithandrea.com%2F "http://codewithandrea.com/") 是主机，向我们展示资源所在的位置。
*   **路径** - 标识用户想要访问的主机中的特定资源。对于我们的示例，`/articles/parse-json-dart/`是路径。

有时，您还会看到这些：

*   **查询参数** - 这些是标记后添加的额外选项`?`。它们通常是帮助排序或过滤数据。
*   **端口** - 用于访问服务器的网络服务。我们经常在开发设置中看到它。如果未提及，HTTP 和 HTTPS 将保留其默认端口（80 和 443）。
*   **片段** - 也称为锚点，此可选位跟随 a`#`并放大网页的特定部分。

这是一个带注释的示例，显示了上述所有内容：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3897787a2eb84d9aa26db9c761379dc0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1506&h=208&s=118237&e=png&b=fffefe) _包含方案、主机、端口、路径、查询参数和片段的完整 URL 示例。_

了解链接的结构将帮助您在 Flutter 应用程序中有效地处理它。如果您处于项目的早期阶段，您可能对如何设计 URL 以使其保持整洁且易于处理有发言权。

请记住，某些 URL 的设置比其他 URL 更好。如果您的 URL 设计不佳，那么在应用程序中处理它们可能会很棘手。请继续阅读本指南的 Flutter 实现部分，了解它是如何完成的。

[深层链接的类型](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23types-of-deep-links "https://codewithandrea.com/articles/flutter-deep-links/#types-of-deep-links")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

移动应用程序可以根据它们使用的方案处理两种类型的深层链接。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d02d0db4bc84755948fdcbe06cbf49f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=953&h=784&s=209067&e=png&b=70d78c)

_深层链接是常规网络链接的超集_

### [定制方案](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23custom-schemes "https://codewithandrea.com/articles/flutter-deep-links/#custom-schemes")

考虑此自定义方案 [URI](https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FUniform_Resource_Identifier "https://en.wikipedia.org/wiki/Uniform_Resource_Identifier") 作为示例：

*   `yourScheme://your-domain.com`

在 Android 中，这称为深层链接，而在 iOS 世界中，这称为自定义 URL 方案。如果您没有域名但想利用深层链接的力量，那么此方法非常方便。

您可以选择任何您喜欢的自定义方案，只要您在应用程序中定义它即可。设置非常快，因为您不必担心唯一性。

但缺点是它不太安全，因为任何应用程序都可能劫持您的自定义方案并尝试打开您的链接。另外，如果有人没有您的应用程序并单击自定义方案链接，他们会陷入错误消息的死胡同。虽然这不是最佳实践，但其简单性使其可以方便地进行快速测试。

### [HTTP/HTTPS 方案](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23httphttps-scheme "https://codewithandrea.com/articles/flutter-deep-links/#httphttps-scheme")

这些是我们每天遇到的常规网址：

*   `https://your-domain.com`

此方法在 Android 上称为 “应用程序链接”，在 iOS 上称为 “通用链接”，它提供了向移动应用程序添加深度链接支持的最安全方法。

它要求您拥有一个域并在两端执行验证。您必须在应用程序代码（Android 上的清单文件和 iOS 上的关联域）中注册您的域，并在服务器端验证您的移动应用程序。

通过执行此舞蹈，您的应用程序可以识别域，并且域可以验证您的应用程序。这种双向验证可确保深层链接的完整性和真实性，提供安全的深层链接体验。👍

理论已经足够了；让我们开始平台设置吧！

[在 Android 上设置深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23setting-up-deep-links-on-android "https://codewithandrea.com/articles/flutter-deep-links/#setting-up-deep-links-on-android")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

让我们从调整`AndroidManifest.xml`文件开始：

*   打开`android/app/src/main/AndroidManifest.xml`文件
*   `activity`向您的标签添加意图过滤器
*   指定您的方案和主机
*   可选：定义支持的路径

您的意图过滤器应如下所示：

```
<intent-filter android:autoVerify="true">
  <action android: />
  <category android: />
  <category android: />

  <data android:scheme="https" />
  <data android:host="yourDomain.com" />
</intent-filter>


```

如果您想使用自定义方案并打开类似 的链接`yourScheme://yourDomain.com`，请使用：

```
<data android:scheme="yourScheme" />
<data android:host="yourDomain.com" />


```

上面的示例没有指出任何特定路径。通过此设置，您的应用程序可以打开您域中的**所有 URL。** 这通常是不可取的，如果您想要更多控制，则需要通过添加相关路径标签来定义接受的路径。最常见的是`path`、`pathPattern` _、_ 和`pathPrefix`。

```
<!-- exact path value must be "/login/" -->
<data android:path="/login/" />  

<!-- path must start with "/product/" and then it can contain any number of characters; in this case, it would probably be a product ID. -->
<data android:pathPattern="/product/.*" />

<!-- path must start with /account/, so it may be /account/confirm, /account/verify etc. -->
<data android:pathPrefix="/account/" />


```

[您可以在 Android 官方文档中](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fmanifest%2Fdata-element "https://developer.android.com/guide/topics/manifest/data-element")找到有关如何使用各种数据标签的详细指南。

### [安卓服务器验证](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23android-server-verification "https://codewithandrea.com/articles/flutter-deep-links/#android-server-verification")

现在应用程序知道它应该处理哪种类型的 URL，您必须确保您的域将您的应用程序识别为受信任的 URL 处理程序。

> 请记住，此步骤仅适用于 HTTP/HTTPS 链接，而不适用于自定义方案。

**步骤 1**：创建一个名为 `assetlinks.json`. 该文件将包含将您的网站与应用程序关联的数字资产链接。这是里面应该包含的内容：

```
[{  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example",
    "sha256_cert_fingerprints":
    ["00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00"]
  }
}]


```

替换 `"com.example"` 为应用程序的包名称，并将该 `sha256_cert_fingerprints` 值替换为应用程序的唯一 SHA256 指纹。

**第 2 步**：将 `assetlinks.json` 文件上传到您的网站。它必须可以通过 访问 `https://yourdomain/.well-known/assetlinks.json`。如果您不知道如何操作，请联系您的域管理员。

**步骤 3**：要检查 `assetlinks.json` 设置是否正确，请使用 Google 提供的语句列表测试器工具，网址为 `https://developers.google.com/digital-asset-links/tools/generator`。

您可以使用以下命令生成应用程序的 SHA256 指纹：

```
keytool -list -v -keystore <keystore path> -alias <key alias> -storepass <store password> -keypass <key password>


```

或者，如果您通过 Google Play 商店对生产应用程序进行签名，则可以在开发者控制台的设置 > 应用程序完整性 > 应用程序签名中找到它。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e634dd8a91d4534a748a461d28c7258~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1816&h=900&s=372289&e=png&b=fefefe) _Google Play Console 上的应用签名页面_

### [如何在 Android 上测试深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23how-to-test-deep-links-on-android "https://codewithandrea.com/articles/flutter-deep-links/#how-to-test-deep-links-on-android")

准备好在 Android 上测试您的深层链接了吗？只需启动 Android 模拟器并弹出终端即可运行这些命令。

对于 HTTP/HTTPS 链接：

```
adb shell am start -a android.intent.action.VIEW \
  -c android.intent.category.BROWSABLE \
  -d https://yourDomain.com


```

对于自定义方案：

```
adb shell am start -a android.intent.action.VIEW \
  -c android.intent.category.BROWSABLE \
  -d yourScheme://yourDomain.com


```

> 小心！ADB 通常位于该`android/sdk/platform-tools`目录中。在运行这些命令之前，请确保它位于您的系统路径中。

> 遇到 “adb：多个设备 / 模拟器” 障碍？如果您有多个 Android 设备或模拟器，请查看[此线程](https://link.juejin.cn?target=https%3A%2F%2Fcloud.typingmind.com%2Fshare%2F40ba089a-d6d6-490a-b762-663bbda0ffd9 "https://cloud.typingmind.com/share/40ba089a-d6d6-490a-b762-663bbda0ffd9")以获取修复。

另一种测试方法是`https://yourDomain.com`使用任何应用程序（例如 WhatsApp）将链接发送给自己，然后点击它。目前，我们的目标只是让应用程序焕发活力。稍后我们将学习如何使用 Flutter 代码导航到正确的屏幕。

🚨 **可能的问题**🚨

如果点击 URL 将在源应用程序（例如 Gmail 或 WhatsApp）“内部” 打开您的应用程序，而不是其自己的窗口，请在标记`android:launchMode="singleTask"`内添加`<activity>`。

以下是使用 WhatsApp 的深层链接打开该应用程序后的外观示例：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f4b87bebbcf47068663a2f5d1ebc1ff~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=588&h=1132&s=231971&e=png&b=fafafa)

_显示目标应用程序在另一个应用程序 (Whatsapp) 中打开的示例_

### [了解消歧对话框](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23understanding-the-disambiguation-dialog "https://codewithandrea.com/articles/flutter-deep-links/#understanding-the-disambiguation-dialog")

消歧对话框是当用户单击多个应用程序可以打开的链接时 Android 会显示的提示。这是系统询问 “哪个应用程序应该处理这个问题？” 的一种方式。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d39f93f2951e4c838049f8ff0ce809bd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=590&h=1132&s=100684&e=png&b=cecece)

_Android 上的消歧对话框示例_

如果您在单击链接后看到此对话框，则说明您的深层链接设置几乎是完美的，但在某处存在一个小问题。您的清单文件中缺少一项常见的疏忽`android:autoVerify="true"`，而这是绕过此对话框的关键。

或者，您的文件可能有问题`assetlinks.json`。仔细检查指纹并查找包名称中是否有任何拼写错误。

解决这些小问题将使用户的旅程更加顺畅，让您的链接直接在应用程序中打开，并完全绕过消歧对话框。

请注意，在调试过程中，此对话框是完全正常的。这是因为该应用程序尚未使用生产密钥进行签名，因此它不会与`assetlinks.json`.

[在 iOS 上设置深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23setting-up-deep-links-on-ios "https://codewithandrea.com/articles/flutter-deep-links/#setting-up-deep-links-on-ios")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

切换到 iOS，设置深度链接的过程与 Android 相比有很大不同。

首先，让我们设置您的关联域。就是这样：

*   在 Xcode 中启动`ios/Runner.xcworkspace`。
*   前往跑步者 > 签名和功能 > + 功能 > 关联域。
*   添加域名`applinks:`时添加前缀。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4285ef8d5b0548f49b1ab7c534ba8371~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1558&h=494&s=126730&e=png&b=fefefe)

_我们可以在 Xcode 的 “签名和功能” 选项卡中添加关联的域_

如果您想使用自定义方案，您还需要在 Xcode 中定义它。这是练习：

*   打开`Info.plist`文件。
*   点击信息属性列表旁边的 + 按钮添加 URL 类型。
*   在其下，弹出 URL 方案并替换`yourScheme`为您正在关注的自定义方案。
*   设置一个标识符（尽管它是什么并不重要）。

完成后，您的`Info.plist`文件应如下所示：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/407367a2877d49b38f3a70ae35593a7d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1148&h=332&s=75351&e=png&b=fdfdfd) _Xcode 中 Info.plist 文件中定义的自定义 URL 方案_

或者，您可以将`Info.plist`文件作为源代码打开并粘贴以下代码段：

```
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>yourScheme</string>
    </array>
    <key>CFBundleURLName</key>
    <string>yourIdentifier</string>
  </dict>
</array>


```

> 请记住，您只需要关注自定义方案的 URL 类型。如果您想要 HTTP/HTTPS 深层链接，则可以跳过这一部分。

### [验证 iOS 服务器的深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23verifying-the-ios-server-for-deep-links "https://codewithandrea.com/articles/flutter-deep-links/#verifying-the-ios-server-for-deep-links")

与 Android 相反，在 iOS 上，服务器上托管的文件不仅包含应用程序信息，还包含支持的路径。以下是如何完成它：

**步骤 1**：创建一个新文件，命名 `apple-app-site-association` 为无文件扩展名。该文件将包含将您的网站和应用程序绑定在一起的数字资产链接。用这样的东西填充它：

```
{
  "applinks":{
    "apps":[
       
    ],
    "details":[
      {
        "appIDs":[
          "TeamID.BundleID"
        ],
        "components":[
          {
            "/":"/login",
            "comment":"Matches URL with a path /login"
          },
          {
            "/":"/product/*",
            "comment":"Matches any URL with a path that starts with /product/."
          },
          {
            "/":"/secret",
            "exclude":true,
            "comment":"Matches URL with a path /secret and instructs the system not to open it as a universal link."
          }
        ]
      }
    ]
  }
}


```

更改`"TeamID.BundleID"`您的应用程序的团队 ID 和捆绑包 ID，并更新`components`以匹配您的应用程序将处理的路径。

iOS 允许您排除路径，因此很常见的是看到排除路径的列表，最后是 “接受所有路径” 定义（因为检查是从上到下完成的）：

```
{
   "/":"*"
}


```

**第 2 步**：将`apple-app-site-association`文件托管在您的网站上。它必须可以通过 访问`https://yourDomain.com/.well-known/apple-app-site-association`。

该 `apple-app-site-association` 文件必须以内容类型提供 `application/json` （但**没有**文件`.json`扩展名！），并且不应签名或加密。

> **注意**`apple-app-site-association`：  Apple 端可能不会立即更改文件 。如果它不能立即起作用，请等待一段时间，然后重试。您可以通过点击 来验证 Apple 是否拥有您的文件的最新版本`https://app-site-association.cdn-apple.com/a/v1/yourDomain.com`。这将返回`apple-app-site-assocation`Apple 验证服务所看到的文件。

### [如何在 iOS 上测试深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23how-to-test-deep-links-on-ios "https://codewithandrea.com/articles/flutter-deep-links/#how-to-test-deep-links-on-ios")

要在 iOS 上测试深层链接，您只需点击链接即可，也可以使用终端使用以下命令进行测试：

对于 HTTP/HTTPS 链接：

```
xcrun simctl openurl booted https://yourDomain.com/path


```

对于自定义方案链接：

```
xcrun simctl openurl booted yourScheme://yourDomain.com/path


```

> 如果您的网站只能通过 VPN（或仅您的登台 / 测试环境）访问，请按照[以下有关设置备用模式的说明](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.apple.com%2Fdocumentation%2Fbundleresources%2Fentitlements%2Fcom_apple_developer_associated-domains "https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains")进行操作。

> **注意**：iOS 的宽松程度比 Android 低得多。如果您的网站上没有该`apple-app-site-association`文件，无论您是直接单击还是使用终端，HTTP/HTTPS 链接都将无法工作。因此，您有两个选择：赶快发布该文件或实施自定义方案以进行测试。

这样，本机设置就完成了！您的应用程序现在应该通过深层链接启动。下一步是将各个点连接起来，确保应用程序不仅可以打开，而且还**可以将用户直接带到他们想要的内容**。这就是 Flutter 发挥作用的地方！

[在 Flutter 中使用 GoRouter 处理深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23handling-deep-links-with-gorouter-in-flutter "https://codewithandrea.com/articles/flutter-deep-links/#handling-deep-links-with-gorouter-in-flutter")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[GoRouter](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Fgo_router "https://pub.dev/packages/go_router") 是许多 Flutter 开发者在导航方面的首选。它使用 Router API 提供基于 URL 的屏幕间导航方法，这与深度链接完美结合。

为了演示如何使用 GoRouter 实现深度链接并指导您完成所有步骤，我创建了一个简单的应用程序（完整的源代码可[在此处获取](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fbizz84%2Fgo_router_deep_links_example "https://github.com/bizz84/go_router_deep_links_example")）。

首先，像这样设置您的应用程序`MaterialApp.router`：

```
void main() {
  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: goRouter,
    );
  }
}


```

然后，创建一个`GoRouter`实例来概述应用程序的路线。让我们从两个简单的屏幕开始：

*   作为应用程序入口点的主屏幕，对应于`https://yourdomain.com`.
*   带有类似 的 URL 的详细信息屏幕`https://yourdomain.com/details/itemId`。

```
final goRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const MainScreen(),
      routes: [
        GoRoute(
          path: 'details/:itemId',
          builder: (context, state) =>
              DetailsScreen(id: state.pathParameters['itemId']!),
        )
      ],
    )
  ],
);


```

请注意，详细信息屏幕嵌套在主屏幕下方。因此，当您导航到详细信息页面时，主屏幕将位于其下方的导航堆栈中。

> `:itemId`路径中的部分表示该 URL 需要一个参数，`/details/`可以使用 来访问`state.pathParameters['itemId']`。

> 您还可以使用 - 获取查询参数`state.uri.queryParameters['paramName']`- 但请注意，GoRouter 10.0.0 更改了查询参数语法。如果您使用的是旧版本，请务必查看此[迁移指南](https://link.juejin.cn?target=https%3A%2F%2Fdocs.google.com%2Fdocument%2Fd%2F1vjupshmFJtfGSOppZxp7Tzkq7dotcLxCcpdluuNYe1o%2Fedit%3Fresourcekey%3D0-aS66t4OcDTjJW50s-veSzQ "https://docs.google.com/document/d/1vjupshmFJtfGSOppZxp7Tzkq7dotcLxCcpdluuNYe1o/edit?resourcekey=0-aS66t4OcDTjJW50s-veSzQ")。

该应用程序如下所示：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6d5440685824eb38c6d3763d0ef7e55~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1640&h=1330&s=242256&e=png&b=fefafe)

_具有列表视图和详细信息屏幕的示例深度链接应用程序_

### [导航到深层链接目标](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23navigating-to-the-deep-link-destination "https://codewithandrea.com/articles/flutter-deep-links/#navigating-to-the-deep-link-destination")

现在路线已经设置完毕，我们需要处理深层链接并将我们的应用程序引向正确的方向。

在网络上，这是开箱即用的。您可以在浏览器中更改 URL，它应该导航到应用程序中的正确位置。

> **Flutter Web 应用程序的一个小提示**：您会注意到路径通常带有哈希片段（如`https://yourDomain.com/#/somePath`）。如果您喜欢不带 的更简洁的外观`#`，则可以在运行应用程序之前调用`usePathUrlStrategy()`您的方法。`main`

对于 Android，您必须将一些元数据放入标签`AndroidManifest.xml`内`activity`：

```
<meta-data
    android:
    android:value="true" />


```

在 iOS 上，您需要将此代码段添加到您的`Info.plist`：

```
<key>FlutterDeepLinkingEnabled</key>
<true/>


```

通过这些调整，您的应用程序将全部设置为响应深层链接，无论它是刚刚启动还是已经运行。

是时候试驾一下了吗？您可以用于`adb`Android 或`xcrun`iOS，就像我们之前讨论的那样：

```
# Test deep link on Android
adb shell am start -a android.intent.action.VIEW \
  -c android.intent.category.BROWSABLE \
  -d https://yourDomain.com/details/3
# Test deep link on iOS
xcrun simctl openurl booted https://yourDomain.com/details/3


```

运行这些命令后，您的应用程序应导航到与深层链接 URL 关联的屏幕。

### [使用 GoRouter 实现重定向和防护](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23implementing-redirects-and-guards-with-gorouter "https://codewithandrea.com/articles/flutter-deep-links/#implementing-redirects-and-guards-with-gorouter")

GoRouter 提供了一种简单的方法来设置用户重定向或 “防护”，正如其他路由包中常见的那样。通过在对象内添加重定向，可以在路由级别实现此功能`GoRoute`：

```
GoRoute(
  path: '/home',
  // always redirect to '/' if '/home' is requested
  redirect: (context, state) => '/',
)


```

当导航事件即将显示路线时，会触发此重定向。

此外，您还可以定义顶级重定向，这对于控制对应用程序某些区域的访问特别有用，例如确保用户在继续操作之前已登录：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cd3eb5a87bf46ed819d985c32bad075~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1640&h=870&s=189547&e=png&b=fffbff)

_具有登录屏幕、列表视图和详细信息屏幕的示例深层链接应用程序_

为了将其付诸实践，您需要一种方法来检查用户是否使用您选择的任何状态管理解决方案登录。

为了简单起见，让我们使用 [Riverpod](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Fflutter_riverpod "https://pub.dev/packages/flutter_riverpod") 进行简单的操作`ChangeNotifier`：

```
class UserInfo extends ChangeNotifier {
  bool isLoggedIn = false;

  void logIn() {
    isLoggedIn = true;
    notifyListeners();
  }

  void logOut() {
    isLoggedIn = false;
    notifyListeners();
  }
}


```

该状态可以存储在`ChangeNotifierProvider`：

```
final userInfoProvider = ChangeNotifierProvider((ref) => UserInfo());


```

现在，让我们`GoRouter`在提供者中定义对象并引入新的`/login`路由：

```
final goRouterProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const MainScreen(),
        routes: [
          GoRoute(
            path: 'details/:itemId',
            builder: (context, state) =>
              DetailsScreen(id: state.pathParameters['itemId']!),
          )
        ],
      ),
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginScreen(),
      ),
    ],
    redirect: (context, state) {
      // Here, you'll add your redirect logic
    },
  );
}


```

将您的顶级小部件转换为 a`ConsumerWidget`并观察方法`goRouterProvider`内部`build`：

```
class App extends ConsumerWidget {
  App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final goRouter = ref.watch(goRouterProvider);
    return MaterialApp.router(
      routerConfig: goRouter,
      ...,
    );
  }
}


```

接下来，在您的 GoRouter 配置中包含顶级重定向。`null`如果不需要重定向，则此函数应返回；如果需要重定向，则应返回新路由：

```
final goRouterProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    routes: [...],
    redirect: (context, state) {
      final isLoggedIn = ref.read(userInfoProvider).isLoggedIn;
      if (isLoggedIn) {
        return null;
      } else {
        return '/login';
      }
    },
  );
});


```

最初，用户会发现自己已注销。他们可以使用简单登录屏幕上的按钮登录：

```
class LoginScreen extends ConsumerWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Welcome!'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            ref.read(userInfoProvider).logIn();
            // explicitly navigate to the home page
            context.go('/');
          },
          child: const Text('Login'),
        ),
      ),
    );
  }
}


```

此处，`context.go('/')`用于将用户从登录导航到主屏幕。然而，这种方法不灵活并且容易出错，尤其是在具有许多路由的大型应用程序上。

### [使用 refreshListenable 刷新 GoRouter](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23refreshing-gorouter-with-refreshlistenable "https://codewithandrea.com/articles/flutter-deep-links/#refreshing-gorouter-with-refreshlistenable")

如果我们可以让 GoRouter 在用户登录时自动更新，而无需手动导航，会怎么样？

好吧，GoRouter 想到了这一点。

它允许您使用参数连接 a `Listenable`（`UserInfo`调用 的类`notifyListeners()`）`refreshListenable`，以便任何状态更改都可以提示刷新。当刷新发生时，更新后的重定向函数可以处理用户位于登录页面但已通过身份验证的情况：

```
final goRouterProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    routes: [...],
    refreshListenable: ref.read(userInfoProvider),
    redirect: (context, state) {
      final isLoggedIn = ref.read(userInfoProvider).isLoggedIn;
      final isLoggingIn = state.matchedLocation == '/login';

      if (!isLoggedIn) return isLoggingIn ? null : '/login';
      if (isLoggingIn) return '/';

      return null;
    },
  );
});


```

> **注意**：在上面的代码片段中，我们使用`ref.read`而不是`ref.watch`因为我们不希望`GoRouter`在身份验证状态更改时重建整个内容。相反，GoRouter 将在（可监听）状态发生变化时在内部进行检查，并`redirect`根据需要调用回调。

完成此操作后，登录后不再需要手动导航；GoRouter 会为您完成繁重的工作。

> 如果您使用不同的状态管理技术并依赖于流（例如 flutter_bloc），则可以通过使用`GoRouterRefreshStream`如下所示的自定义类来调整它`refreshListenable: GoRouterRefreshStream(streamController.stream)`：有关更多详细信息，请参阅此[迁移指南](https://link.juejin.cn?target=https%3A%2F%2Fdocs.google.com%2Fdocument%2Fd%2F10l22o4ml4Ss83UyzqUC8_xYOv_QjZEi80lJDNE4q7wM%2Fedit "https://docs.google.com/document/d/10l22o4ml4Ss83UyzqUC8_xYOv_QjZEi80lJDNE4q7wM/edit")。

### [带查询参数的基于 URL 的重定向](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23url-based-redirection-with-query-parameters "https://codewithandrea.com/articles/flutter-deep-links/#url-based-redirection-with-query-parameters")

设置防护时，请考虑引导用户返回登录后最初目标的页面。这涉及跟踪用户想要到达的初始位置。您可以将此位置作为参数传递到登录屏幕，或将其存储在类似结构`UserInfo`或与深层链接相关的结构中。

然而，一种更简单、更优雅的方法是通过向重定向添加查询参数来利用基于 URL 的导航。其操作方法如下：

```
redirect: (context, state) {
  final isLoggedIn = ref.read(userInfoProvider).isLoggedIn;
  final isLoggingIn = state.matchedLocation == '/login';

  final savedLocation = state.matchedLocation == '/' ? '' : '?from=${state.matchedLocation}';

  if (!isLoggedIn) return isLoggingIn ? null : '/login$savedLocation';

  if (isLoggingIn) return state.uri.queryParameters['from'] ?? '/';

  return null;
},


```

此代码检查用户是否已登录以及他们正在导航的位置。如果他们需要登录，它会将他们重定向到登录页面，并将初始位置附加为查询参数。登录后，它会将用户重定向到其初始页面，如果未指定初始页面，则返回到主页。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b9ef87233704852bace07b791a4f4b4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1640&h=870&s=163024&e=png&b=fffbff)

_示例显示带有查询参数的基于 URL 的导航如何首先将用户引导至登录屏幕，然后引导至目标目的地（通过列表视图跳过主页）_

如果您使用 URL 打开应用程序`${yourScheme}://${yourDomain}/details/4`，您将首先点击登录屏幕，然后看到第 4 项的详细信息。

但是，如果有人尝试访问不存在的路由（例如`https://yourdomain.com/register`或 ）怎么办`https://yourdomain.com/details/15001900`？这是一个很好的问题！

### [通过错误处理解决不正确的路径](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23tackling-incorrect-paths-with-error-handling "https://codewithandrea.com/articles/flutter-deep-links/#tackling-incorrect-paths-with-error-handling")

当您深入了解深层链接错误处理时，您需要记住几种类型的错误。

**在路径定义级别**

`AndroidManifest.xml`您可以在文件（对于 Android）和文件（对于 iOS）中定义接受的路径`apple-app-site-association`。如果用户尝试不适合这些模式的路径，他们的浏览器将打开它，而不是您的应用程序。在这种情况下，您的 Flutter 代码不需要执行任何操作。

**在路径匹配级别**

仅仅因为深层链接会唤醒您的应用程序，并不一定意味着该路径是有效的。也许您已经有了 “打开所有路径” 配置，但您仍然需要验证 GoRouter 设置中的每个路径。

如果深层链接指向您的应用中不存在的路线，您应该提供明确的错误消息或一般后备屏幕。

这是当路由丢失时默认情况下使用 GoRouter 得到的结果（这是我尝试时弹出的结果`https://yourdomain.com/register`）：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fecb7cbcfa64933a7794717d0fba9ce~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=758&h=1342&s=115661&e=png&a=1&b=fffbff) _缺少路由时显示默认页面未找到屏幕_

这并不是最友好的页面，对吧？`errorPageBuilder`您可以使用或 GoRouter 参数对其进行自定义`errorWidgetBuilder`。或者您可以使用`onException`处理程序，您可以在其中选择重定向用户，就这样吧，等等。

**在内容层面**

太棒了！你们的道路很匹配，感觉就像是一个小小的胜利。但坚持住！正确的路径并不意味着用户想要的项目（例如特定的产品 ID 或书名）实际上存在于您的数据库中。

如果内容缺失，则需要优雅地处理。发布消息或 “未找到内容” 页面，让用户随时了解情况。目的是吸引他们并提供无缝体验，即使出现问题也是如此。

[结论](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23conclusion "https://codewithandrea.com/articles/flutter-deep-links/#conclusion")
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

深层链接可以通过提供到应用程序内特定区域的直接通道来显着增强用户体验。本指南提供了全面的概述，从设置本机 Android 和 iOS 代码到使用 GoRouter 管理路由。

您的反馈非常宝贵。如果您有任何问题或特别感兴趣的领域，请随时分享。随着 Flutter 深度链接之旅的继续，您的好奇心将有助于塑造对未来主题的探索。

但是等等，深度链接的故事还有更多内容。一些其他主题和工具值得进一步探索。👇

### [外部深度链接包](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23external-deep-linking-packages "https://codewithandrea.com/articles/flutter-deep-links/#external-deep-linking-packages")

app_links 和 [uni_links](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Funi_links "https://pub.dev/packages/uni_links") 包提供了处理 Flutter 中深度链接的附加功能[。](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Fapp_links "https://pub.dev/packages/app_links")如果您需要使用复杂的 URL 模式，它们可能特别有用 - 在 GoRouter 配置中处理它们很快就会变得很麻烦。[](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Funi_links "https://pub.dev/packages/uni_links")

这些软件包为您提供完全的控制权：您只需接收一个 URL 并可以将其映射到您喜欢的任何屏幕或操作。但是，就像在生活中一样，权力越大，责任也越大。您必须自行处理所有路径匹配和参数提取。

### [延迟深层链接](https://link.juejin.cn?target=https%3A%2F%2Fcodewithandrea.com%2Farticles%2Fflutter-deep-links%2F%23deferred-deep-links "https://codewithandrea.com/articles/flutter-deep-links/#deferred-deep-links")

我们不要忘记延迟的深层链接。这对于通过广告推广的应用程序来说是一个福音，使新用户能够在通过链接安装应用程序后立即登陆相关页面。此功能的提供商通常会捆绑高级营销和分析工具，以帮助您跟踪转化、归因和用户来源。最流行的解决方案包括 [branch.io](https://link.juejin.cn?target=https%3A%2F%2Fwww.branch.io%2F "https://www.branch.io/")、[Adjust](https://link.juejin.cn?target=https%3A%2F%2Fwww.adjust.com%2F "https://www.adjust.com/")、[AppsFlyer](https://link.juejin.cn?target=https%3A%2F%2Fwww.appsflyer.com%2F "https://www.appsflyer.com/") 和 [Kochava](https://link.juejin.cn?target=https%3A%2F%2Fwww.kochava.com%2F "https://www.kochava.com/")。