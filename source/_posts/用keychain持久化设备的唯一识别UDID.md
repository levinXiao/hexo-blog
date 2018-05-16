---
title: 用keychain持久化设备的唯一识别UDID
date: 2016-09-28 09:50:11
tags: [iOS]
categories: iOS
---

在iOS中,如果想获得手机的唯一标识UDID,与硬件相关而与软件无关,在Apple的API中是不提供的

# CFUUID
每次调用 CFUUIDCreate 系统都会返回一个全新的唯一 ID. 如果想永久保存这个 ID，需要自己处理，可以一次获取后，存在 NSUserDefaults，Keychain，Pasteboard 等，下次再从这其中取出。

<!-- more -->

```
- (NSString *)createUUID {
    CFUUIDRef uuid = CFUUIDCreate(NULL);
    CFStringRef string = CFUUIDCreateString(NULL, uuid);
    CFRelease(uuid);
    return (__bridge NSString *)string;
}
```

# NSUUID

```
NSString *uuid = [[NSUUID UUID] UUIDString];
```
和 CFUUID 一样, 每次调用 NSUUID 都会获取到一个新的 UUID，需要自己保存获取到的 UUID。

# Advertiser Identifier

```
NSString *adId = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
```
**AdvertisingIdentifier** 由系统保持唯一性，同一个设备上所有应用获取到 advertisingIdentifier 的都是一样的。但有两种方法可以修改这个值，重置系统（设置 -> 通用 -> 还原 -> 抹掉所有内容和设置）或者重置广告标识符（设置 -> 隐私 -> 广告 -> 还原广告标识符），在这两种情况下都会生成新的**AdvertisingIdentifier**，依然无法保证设备唯一。

# Identifier for Vendor (IDFV)
在Apple的API中,有一个**Identifier for Vendor (IDFV)**的方法

```
NSString *idfv = [[[UIDevice currentDevice] identifierForVendor] UUIDString];
```

在同一个设备上不同的 vendor 下的应用获取到的 IDFV 是不一样的，而同一个 vendor 下的不同应用获取的 IDFV 都是一样的。但如果用户删除了这个 vendor 的所有应用，再重新安装它们，IDFV 就会被重置，和之前的不一样，也不是设备唯一的。

什么是 Vendor？ 一个 Vendor 是 CFBundleIdentifier——应用标识符的前缀

举个例子， com.doubleencore.app1 和 com.doubleencore.app2 会获取到相同的 IDFV，因为它们有相通的 com.doubleencore。但 com.massivelyoverrated 和 net.doubleencore 获取到的则是不一样的 IDFV。

以上所有 ID 都不能保证设备唯一，有什么方式可以获取设备唯一 ID？

以 IDFV 为例，为保证用户在删除应用时，取到的 IDFV 仍和之前的一样，可以借助 Keychain。使用 SAMKeychain，可以很方便地设置 keychain。 需要注意的是， keychain 在同一个苹果账号的不同设备下是同步的，需要设置query.synchronizationMode = SAMKeychainQuerySynchronizationModeNo;，不在设备间同步这个值，这样不同设备获取到的便是不同的 ID，代码如下：

```
-(NSString *)getUniqueDeviceIdentifierAsString {
    NSString *appName=[[[NSBundle mainBundle] infoDictionary] objectForKey:(NSString*)kCFBundleNameKey];

    NSString *strApplicationUUID = [SAMKeychain passwordForService:appName account:@"incoding"];
    if (strApplicationUUID == nil) {
        strApplicationUUID  = [[[UIDevice currentDevice] identifierForVendor] UUIDString];

        NSError *error = nil;
        SAMKeychainQuery *query = [[SAMKeychainQuery alloc] init];
        query.service = appName;
        query.account = @"incoding";
        query.password = strApplicationUUID;
        query.synchronizationMode = SAMKeychainQuerySynchronizationModeNo;
        [query save:&error];
    }
    return strApplicationUUID;
}
```
