---
title: 生物认证 TouchID/FaceID
date: 2022-06-30

tags: ["iOS", "FaceID", "TouchID"]
series: ["IOS"]
series_order: 3
---

# 事前准备

添加权限请求描述: e.g. "App需要TouchID或FaceID权限，来验证使用者是否是手机所有者"
```
Privacy - Face ID Usage Description
```
OR
```
NSFaceIDUsageDescription
```

引入相关的库

```
LocalAuthentication
```
![LocalAuthentication](../images//LocalAuthentication.png)

引入头文件
```objectivec
#import <LocalAuthentication/LocalAuthentication.h>
```

# 使用

```objc
- (void)faceID {
    
    LAContext *context = [LAContext new];
    context.localizedFallbackTitle = @"localizedFallbackTitle";
    NSError *error;
    LAPolicy policy = LAPolicyDeviceOwnerAuthenticationWithBiometrics;
    if ([context canEvaluatePolicy:policy error:&error]) {
        [context evaluatePolicy:policy localizedReason:@"通过Home键验证已有指纹" reply:^(BOOL success, NSError * _Nullable error) {
                    if (success) {
                        // success
                        MNLog(self, @" successed!!!");
                    } else if (error) {
                        MNLog(self, @" face error.code = %@", error);
                        switch (error.code) {
                            case LAErrorAuthenticationFailed:
                                break;
                            default:
                                break;
                        }
                    } else {
                        // other
                        MNLog(self, @" face id the other reason!!!");
                    }
        }];
    } else {
        // error
        MNLog(self, @"还没有设置id, %@", error);
    }
    
}
```