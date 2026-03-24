---
description: 为了进一步简化用户对接流程，提高算法质量，共享固件资源，将公版app所用的服务进行共享，包含睡眠算法，固件升级，timeline，血压血糖算法等
icon: square-rss
---

# 获取token

### 申请key

合作方可以联系我们，提供贵公司的名称，我们分配调用服务的key

### android额外配置

服务的接口是http的，如果调用方使用的是https，需要同时兼容两种模式，如调用服务无响应，可能是因为CLEARTEXT communication to XX not permitted by network security policy 这样的错误

### 申请token

根据申请的key，和使用sdk的用户的唯一id（比如手机号，用户名，身份证号，id等），就可以申请token，token会自动保存在本地，不需要用户保存，尽量在合适的时候（比如每次app启动时），调用这个接口，刷新一下Token，防止被踢或者Token过期

**android：**

```java
LogicalApi:
  public static void createToken(String key,String userName, ICreateToken iCreateToken)

  LogicalApi.createToken("","", new ICreateToken() {
            @Override
            public void getTokenSuccess() {

            }

            @Override
            public void error(String msg) {

            }
        });
```
### 创建Token

**iOS:**
```Swift
/// 创建Token
/// - Parameters:
///   - apiKey: API密钥
///   - userIdentifier: 用户标识
/// - Parameter completion: 创建Token回调
func createToken(apiKey: String, userIdentifier: String, completion: @escaping (Result<String, BCLError>) -> Void)

/// 刷新Token
/// - Parameter completion: 刷新Token回调
func refreshToken(completion: @escaping (Result<Void, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.createToken(apiKey: "your_api_key", userIdentifier: "user_123") { result in
    switch result {
    case .success(let token):
        print("Token: \(token)")
    case .failure(let error):
        print("创建Token失败: \(error)")
    }
}
```