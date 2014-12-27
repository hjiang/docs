# SNS 组件开发指南


AVOSCloudSNS 是一个非常轻量的模块, 可以用最少一行代码就可以实现社交平台用户登录.


## iOS SNS 组件

### 导入 SDK
您可以从 https://github.com/leancloud/avoscloud-sdk 下载iOS的SDK。您也可以使用Podfile通过cocoapods下载我们的SDK，如:

```sh
pod 'AVOSCloudSNS'
```

### 添加代码

```objc
[AVOSCloudSNS loginWithCallback:^(id object, NSError *error) {
        //you code here
    }];
```

就这么简单, 现在你已经拥有了用户登录功能!

### SSO
利用SSO, 可以使用户不用输入用户名密码等复杂操作, 一键登录. **目前 LeanCloudSNS 已经支持新浪微博和手机QQ, 并且不需要使用各个平台官方的SDK, 保证您的app体积最小化**. 而你需要做的也很简单,以新浪微博为例:

```objc
[AVOSCloudSNS setupPlatform:AVOSCloudSNSSinaWeibo withAppKey:@"Weibo APP ID" andAppSecret:@"Weibo APP KEY" andRedirectURI:@""];

[AVOSCloudSNS loginWithCallback:^(id object, NSError *error) {
        //you code here
    } toPlatform:AVOSCloudSNSSinaWeibo];

```

在AppDelegate里添加:

```objc
-(BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url{
    return [AVOSCloudSNS handleOpenURL:url];
}
```

这样, 代码部分就完成了. 下一步就是设置, 为你的app添加 URL Schemes: `sinaweibosso.微博appId`(注意有个点".")

这时如果顺利, 应该可以正常的打开新浪微博官方iOS客户端进行登录了.

QQ的SSO与微博完全一致, 只是设置URL Schemes:`tencent腾讯appid`

### 绑定 AVUser

先导入头文件:

```objc
# import <AVOSCloudSNS/AVUser+SNS.h>
```

然后在登录SNS成功回调后`loginWithAuthData:block`,这样会用当前的SNS用户来尝试登录获取AVUser信息, 如果AVUser不存在, 系统会自动创建新用户并返回,如果已经存在, 则直接返回该用户.

```objc
[AVUser loginWithAuthData:object block:^(AVUser *user, NSError *error) {
     //返回AVUser
}];
```

如果需要为AVUser,可以用`addAuthData:block:`方法,比如:

```objc
[user addAuthData:object block:^(AVUser *user, NSError *error) {
     //返回AVUser
}];
```

**上述方法同样适用于`AVUser`的子类.**

### 手动显示登录界面
上面的例子中都是自动显示登录界面, 如果需要实现自定义显示方式,可以使用方法`loginManualyWithCallback:`, 请看下面的例子:

```objc
__block UIViewController *vc=nil;
vc= [AVOSCloudSNS loginManualyWithCallback:^(id object, NSError *error) {
    if (vc) {
        //关闭UIViewController

        //ARC模式中要将vc置空
        vc=nil;
    }
}];

if (vc) {
    //显示UIViewController
}
```

请注意代码提到的`ARC模式中要将vc置空`来打破retain环, 否则vc将不会被释放而浪费内存!


更多详细使用方法, 请查看SDK API文档.

## Android SNS 组件


AVOSCloud Android SNS为开发人员提供了一个非常轻量的模块, 使用此模块，您可以仅用少量代码便可实现社交平台用户登录的功能.

### 导入 SDK

您可以从 [本地下载](https://leancloud.cn/docs/sdk_down.html) Android SNS SDK（从1.4.4版本开始包括avossns的jar包），将下载的jar包加入您工程的libs目录。如果您还不知道如何安装SDK，请查看[快速入门指南](/start.html)。

### WebView 授权

首先你需要在[管理界面](https://leancloud.cn/app.html?appid={{appid}}#/sns)中间配置相应平台的AppKey与AppSecret,在成功保存以后，页面上能够得到相应的`回调URL`和`登录URL`。你将在代码里用到`登陆 URL`,同时请将`回调URL`填写到对应平台的App管理中心（比如新浪开放平台）

之后你需要在AndroidManifest.xml中间添加相应的Activity:

```
        <activity
            android:name="com.avos.sns.SNSWebActivity" >
        </activity>
```

同时你需要拷贝我们准备的res/avoscloud_sns_web_activity.xml到你的项目中去。

之后在你需要授权的地方，你就可以通过WebView进行相应的授权：

```
   public class AuthActivity extends Activity{

   public void onCreate(){

          SNS.setupPlatform(SNSType.AVOSCloudSNSSinaWeibo,
              "登录URL");
          SNS.loginWithCallback(getActivity(), SNSType.AVOSCloudSNSSinaWeibo, new SNSCallback() {

            @Override
            public void done(SNSBase base, SNSException e) {
              if(e==null){
               SNS.loginWithAuthData(base.userInfo(), new LogInCallback<AVUser>() {

                  @Override
                  public void done(final AVUser user, AVException e) {
                  }
              });
              }
            }
          });
   }

   @Override
   protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        SNS.onActivityResult(requestCode, resultCode, data, type);
    }
}

```

这样就完成了新浪微博从授权到创建用户（登录）的一整套流程。
**注:由于微信的Web授权页面为扫描二维玛，现阶段仅仅支持新浪微博和QQ授权。**

### SSO 登录

利用SSO, 用户可以使用单点登录功能，避免反复输入用户名和密码等，您可以使用以下代码轻松实现:

```java
// 导入sns组件
import com.avos.sns.*;

// 使用新浪微博SNS登录，在您的Activity中
public class MyActivity extends Activity {

    // onCreate中初始化，并且登录
    public void onCreate(Bundle savedInstanceState) {
        ….
        // callback 函数
		final SNSCallback myCallback = new SNSCallback() {
            @Override
            public void done(SNSBase object, SNSException e) {
                if (e == null) {
                    showText("login ok " + type );
                }
            }
        };

        // 关联
        SNS.setupPlatform(SNSType.AVOSCloudSNSSinaWeibo, "YOUR_SINA_WEIBO_APP_ID", "", "YOUR_SINAWEIBO_CALLBACK_URL");
		SNS.loginWithCallback(this, SNSType.AVOSCloudSNSSinaWeibo, myCallback);
    }

    // 当登录完成后，请调用SNS.onActivityResult(requestCode, resultCode, data, type);
    // 这样您的回调用将会被调用到
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        SNS.onActivityResult(requestCode, resultCode, data, type);
    }

```
您也可以将上述代码中的SNSType.AVOSCloudSNSSinaWeibo 更换为 SNSType.AVOSCloudSNSQQ 便可以使用QQ的SSO登录功能。

** QQ SNS 在腾讯SNS授权中，由于QQ SDK官方对于WebView授权的限制，导致在WebView中无法完成正常的授权过程，所以QQ SNS只支持SSO登陆授权。我们也会持续跟进QQ SDK的更新进展，同时也为对您造成的不便感到抱歉。

***QQ SSO注意*** 当使用QQ SSO登陆时，请注意确保您的AndroidManifest.xml文件中包含如下内容

```xml
<activity android:name="com.tencent.tauth.AuthActivity"
          android:noHistory="true"
          android:launchMode="singleTask">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
    </intent-filter>
</activity>
```

对于其他的sso请确保您有如下权限

```xml
</application>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
</manifest>
```

### 绑定 LeanCloud User

您也可以将SSO登录后的帐号信息与LeanCloud 的User绑定，通过绑定，您可以在两种用户体系间建立联系，方便信息的共享和使用。

如果您还未安装LeanCloud Android SDK，请参阅[快速入门指南](/start.html)。

SSO登录过程与上述代码都相同，您只需要在callback中进行关联即可，示例代码如下

```java
final SNSCallback myCallback = new SNSCallback() {
     @Override
     public void done(SNSBase object, SNSException e) {
         if (e == null) {
            SNS.associateWithAuthData(AVUser.getCurrentUser(), object.userInfo(), null);
         }
     }
 };
```

上述代码，可以将您的SNS帐号与已经创建的LeanCloud User帐号绑定。您也可以使用 `loginWithAuthData`，它将为您创建一个新的匿名用户

```java
  SNS.loginWithAuthData(authData, new LogInCallback() {
    @Override
    public void done(AVUser avUser, AVException e) {
        if (e == null) {
            MyActivity.this.showText("create new user with auth data done");
        } else {
            MyActivity.this.showText("create new user with auth data error: " + e.getMessage());
        }
    }
});
```

如果您不想再使用SNS相关的功能，您可以使用 `logout` 解除SNS帐号和LeanCloud User帐号的绑定。

```java
  SNS.logout(AVUser.getCurrentUser(), type, new SaveCallback() {
      @Override
      public void done(AVException e) {
      }
  });
```


更多详细使用方法, 请查看SDK API文档.

#### 不引入SNS模块的第三方账号与AVUser绑定

在实际的使用过程中，有一部分用户在涉及到SNS相关功能，比如分享模块时，引用了其他第三方库，然而这些库中所引用的SNS jar很有可能与LeanCloud存在版本的冲突。这个时候，用户往往非常的苦恼，无法解决这样的问题。考虑到这一部分用户的需求，我们在AVUser中间添加了几个类似的方法，以便用户能够更便捷地与AVUser进行绑定，从而快速的结合LeanCloud的用户系统。

通过`AVUser.loginWithAuthData`来创建一个匿名的AVUser对象：
```java
    AVUser.AVThirdPartyUserAuth userAuth = new AVUser.AVThirdPartyUserAuth(accessToken, expiresAt, snsType);//此处snsType 可以是"qq","weibo"等字符串
    AVUser.loginWithAuthData(clazz, userAuth, callback);
```

或者通过`AVUser.associateWithAuthData`或者`AVUser.dissociateAuthData`来为一个已经存在的AVUser对象来绑定一个第三方账号或者解除第三方账号绑定：

```java
    AVUser.AVThirdPartyUserAuth userAuth = new AVUser.AVThirdPartyUserAuth(accessToken, expiresAt, snsType)
    AVUser.associateWithAuthData(AVUser.getCurrentUser(),userAuth,callback);

    AVUser.dissociateAuthData(AVUser.getCurrentUser(),AVThirdPartyUserAuth.SNS_TENCENT_WEIBO,callback);// 解除腾讯微薄的账号绑定
```

如果您使用了ShareSDK来实现分享和第三方登陆的功能，能也可以非常轻松的完成和AVUser的绑定。以新浪微博为例

```java
   Platfrom plat = new SinaWeibo(getActivity());
   if (plat.isValid()){
   //如果已经授权过了，直接拿出来用
        String userId = plat.getDb().getUserId();
        if (!Utils.isNullOrEmpty(userId)) {

        //绑定第三方的授权信息
          AVUser.AVThirdPartyUserAuth auth =
              new AVUser.AVThirdPartyUserAuth(plat.getDb().getToken(), String.valueOf(plat.getDb()
                  .getExpiresTime()), AVUser.AVThirdPartyUserAuth.SNS_SINA_WEIBO, plat.getDb()
                  .getUserId());
          AVUser.loginWithAuthData(auth, new LogInCallback<AVUser>() {

            @Override
            public void done(AVUser user, AVException e) {
              if (e == null) {
              //恭喜你，已经和我们的AVUser绑定成功
              } else {
                e.printStackTrace();
              }
            }
          });
          return;
        }
   }
      //如果尚未授权，则通过ShareSDK完成授权
      plat.setPlatformActionListener(new PlatformActionListener() {

        @Override
        public void onError(Platform arg0, int arg1, Throwable arg2) {

        }

        @Override
        public void onComplete(Platform plat, int arg1, HashMap<String, Object> arg2) {
        //授权已经完成
        //绑定第三方的授权信息
          AVUser.AVThirdPartyUserAuth auth =
              new AVUser.AVThirdPartyUserAuth(plat.getDb().getToken(), String.valueOf(plat.getDb()
                  .getExpiresTime()),AVUser.AVThirdPartyUserAuth.SNS_SINA_WEIBO , plat.getDb()
                  .getUserId());
          AVUser.loginWithAuthData(auth, new LogInCallback<AVUser>() {

            @Override
            public void done(AVUser user, AVException e) {
              if (e == null) {
              //恭喜你，已经和我们的AVUser绑定成功
              } else {
                e.printStackTrace();
              }
            }
          });
        }

        @Override
        public void onCancel(Platform arg0, int arg1) {

        }
      });
      plat.SSOSetting(true);
      plat.showUser(null);
```
