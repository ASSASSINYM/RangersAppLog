# RangersAppLog

数据采集上报SDK。具体支持功能见官网 https://datarangers.com.cn/

本仓库包含Xcode Demo工程和6.0.0以下版本的SDK发布源。6.0.0及以上版本发布在[火山引擎Cocoapods Spec源](https://github.com/volcengine/volcengine-specs/tree/master/RangersAppLog)，不发布在本仓库。

## Demo演示

**Example工程1**
1. `git clone git@github.com:bytedance/RangersAppLog.git`
2. `cd RangersAppLog/Example`
3. `pod install`
4. `open Example.xcworkspace`

**Example工程2**
1. `git clone git@github.com:bytedance/RangersAppLog.git`
2. `cd RangersAppLog/ObjCExample`
3. `pod install`
4. `open ObjCExample.xcworkspace`

## 开发环境要求

特别说明，只支持XCode 11+ 打包开发。若使用的Xcode版本为11以下，请单独联系开发提供SDK包

* iOS 9.0+
* XCode 11.0+

## 版本说明

1. Lite版，埋点版本，即 `subspecs => ['Core','Log', 'Host/CN']`,如果需要采集IDFA,子库需要添加`Unique`
3. 圈选版本，即 `subspecs => ['Picker', 'Host/CN']`如果需要采集IDFA,子库需要添加`Unique`

## 集成方式

建议使用Cocoapods接入。可以参照下面的实例和Demo工程中的Podfile。

```ruby
# cdn trunk
source 'https://cdn.cocoapods.org/'
## or use ssh
# source 'git@github.com:CocoaPods/Specs.git'

source 'git@github.com:bytedance/cocoapods_sdk_source_repo.git'
source 'https://github.com/volcengine/volcengine-specs.git'  # 6.0.0+版本发布在volcengine-specs源

# 接入无埋点版本
target 'YourTarget' do
    pod 'RangersAppLog', '~> 5.6.6',:subspecs => [
        'Picker',
        'Unique',  # 若需要采集IDFA，则引入Unique子库
        'Host/CN'  # 若您的APP的数据存储在中国, 则选择 Host/CN。否则请根据地域选择相应 Host 子库
    ]
end

# 接入埋点版本 
target 'YourTarget' do
    pod 'RangersAppLog', '~> 5.6.6',:subspecs => [
      'Core',
      'Log',
      'Unique',  # 若需要采集IDFA，则引入Unique子库
      'Host/CN'  # 若您的APP的数据存储在中国, 则选择 Host/CN。否则请根据地域选择相应 Host 子库
    ]
end
```

## 集成指南

更多接口参见头文件，和Demo工程.

### 初始化SDK

```Objective-C

#import <RangersAppLog/RangersAppLog.h>

- (void)startAppLog {
    BDAutoTrackConfig *config = [BDAutoTrackConfig new];
    config.appID = @"159486";
    config.appName = @"dp_tob_sdk_test2";
    config.channel = @"App Store";
	config.autoTrackEnabled = YES;  // 本地无埋点开关。注意使用无埋点时，必须在远端配置中也打开无埋点开关。
    config.showDebugLog = YES;  // show debug log
    config.logger = ^(NSString * _Nullable log) {
        NSLog(@"%@",log);
    };

    [BDAutoTrack startTrackWithConfig:config];

    NSString *uniqueID = @"12345";  // set UserUniqueID if now is loged in
    [BDAutoTrack setCurrentUserUniqueID:uniqueID];
    [BDAutoTrack eventV3:@"play_video" params:nil];  // 打点
}

```

### 用户态变化

```Objective-C

- (void)logout {
    [self.track clearUserUniqueID];
}

- (void)login {
    /// change to your UserUniqueID
    NSString *uniqueID = @"12345";
    [self.track setCurrentUserUniqueID:uniqueID];
}

```

### 埋点事件上报

```Objective-C

+ (void)eventV3:(NSString *)event params:(NSDictionary *)params {
    [self.track eventV3:event params:params];
}

```

### Scheme上报

```Objective-C

#import <RangersAppLog/RangersAppLog.h>

/// 如果是iOS 13中重写UISceneDelegate的回调，则参考以下code
- (void)scene:(UIScene *)scene openURLContexts:(NSSet<UIOpenURLContext *> *)URLContexts {
    for (UIOpenURLContext *context in URLContexts) {
        NSURL *URL = context.URL;
        if ([[BDAutoTrackSchemeHandler sharedHandler] handleURL:URL appID:@"appid" scene:scene]) {
            continue;
        }

        /// your handle code for the URL
    }
}

/// 如果是系统版本小于iOS 13，需要重写UIApplicationDelegate的回调方法，参考以下code
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
    if ([[BDAutoTrackSchemeHandler sharedHandler] handleURL:url appID:@"appid" scene:nil]) {
        return YES;
    }

    /// your handle code

    return NO;
}

```

## 版本更新记录

### 6.3.0
- Feat: 支持私有化加密依赖注入
- Feat: 支持火山引擎广告分析产品多链接实验技术
- Opt: 优化SDK稳定性

### 6.2.3
- Feat: 支持ALink SDK分享裂变产品功能
- Feat: 支持Apple Search Ads归因

### 6.2.0
- Feat: 支持通过接口设置app四位版本号
- Feat: 暴露SDK字符串版本号获取接口
- Feat: 集成安全反作弊模块
- Feat: 被动启动标记
- Feat: 初始化时增加开关控制是否采集内嵌H5无埋点事件
- Feat: 优化用户维度AB测试 - 添加开关：切换用户时若SSID变化则清除ABVersions缓存(含ExternalVersions)
- Feat: 支持识别5G网络
- Opt: 优化premain时间
- Fix: deep copy 偶现crash
- Fix: 其它Fix若干

### 6.0.0
- Feat: 支持ALink
- Feat: H5Bridge: H5页面支持从原生端上报
- Feat: 支持Profile独立路径接口上报
- Feat: 新增`setCustomHeaderWithDictionary:`接口
- Feat: 支持初始 UserUniqueID
- Fix: 切换用户时AB数据更新不及时的问题

### 5.6.6
- 新增CAID子库。引入CAID子库后自动开启CAID。合规提示：Core中不包含与CAID有关的符号。
- fix: 激活请求Query中缺少IDFA, IDFV等字段信息。

### 5.6.5
- 支持「首次启动事件标记」
- ohayoo预置埋点
- 新增一组初始化后立即获取AB配置缓存的方法。见`BDAutoTrack.h`以`Sync`结尾的ABTest相关方法

### 5.6.4
- 支持bitcode
- 支持platform端属性
- 提供类方法单独初始化和单独启动SDK的接口。
  - `+[BDAutoTrack sharedTrackWithConfig:]` 
  - `+[BDAutoTrack startTrack]`

### 5.6.3
- 上报流量优化
- 支持在开发调试阶段清除缓存。见`BDAutoTrackCacheRemover.h` (生产环境请勿使用)
- bugfix

### 5.6.1
- bugfix: 修复激活url_safe base64相关问题

### 5.6.0（有bug，请使用5.6.1+）

- 新增 profile API. 详见头文件`BDAutoTrack+Profile.h`
- 移除移动端圈选。服务端圈选功能不受影响。建议您在平台的圈选页面上进行圈选，更加快捷方便。

### 5.5.0（有bug，请使用5.6.1+）

- 加密开关支持加密query字段
- 数据库文件夹移动到`Library/`目录并改名
- 上报URL隔离到`Host/XX`子库中
- 适配iOS14注册需求
- 增加用户触点(touchPoint)功能和`setTouchPoint`等接口
- 新增一个Objective-C Example工程

### 5.4.1

- 修复私有化配置且打开加密开关的场景下，客户端尝试解密服务器明文回包从而导致未获取到回包数据的问题

### 5.4.0

- 默认移除IDFA，IDFA放到了子库`Unique`中

### 5.3.1

- 修复UUID变化上报不及时问题
- 新增接口


## 证书

本项目使用[MIT 证书](LICENSE)。详细内容参见[证书](LICENSE)文件。
