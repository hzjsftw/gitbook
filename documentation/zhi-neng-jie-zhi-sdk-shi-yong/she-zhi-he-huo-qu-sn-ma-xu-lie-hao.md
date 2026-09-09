---
description: sdk提供了获取戒指SN码（序列号）的方法
icon: ring-diamond
---

# 获取sn码（序列号）

### android：

```java
   /**
     * 设置序列号
     * @param setSnValueResult 序列号
     * @param isnListener1 回调
     */
    public static void SET_SN(String setSnValueResult,ISNListener isnListener1) 
    
     /**
     * 获取序列号
     * @param isnListener1 回调
     */
    public static void GET_SN(ISNListener isnListener1) 
    
    //回调
    public interface ISNListener {
    /**
     * 获取序列号
     * @param sn
     */
    void getSn(String sn);

    /**
     * 设置序列号（SN码）
     * @param success
     */
    void setSn(boolean success);


}
```

### iOS：

```swift
/// 设置SN码
/// - Parameters:
///   - snCode: SN码字符串
///   - completion: 设置SN码回调
/// - Result: 设置SN码结果
/// - BCLFixtureTestingSetSNCodeResponse: 设置SN码响应模型
///   - status: 状态码（0：失败，1：成功）
///   - isSuccess: 是否设置成功的便捷属性
/// - BCLError: 错误信息
func setSNCode(snCode: String, completion: @escaping (Result<BCLFixtureTestingSetSNCodeResponse, BCLError>) -> Void)

/// 获取SN码
/// - Parameter completion: 获取SN码回调
/// - Result: 获取SN码结果
/// - BCLFixtureTestingGetSNCodeResponse: 获取SN码响应模型，包含snCode属性
/// - BCLError: 错误信息
func getSNCode(completion: @escaping (Result<BCLFixtureTestingGetSNCodeResponse, BCLError>) -> Void)
```
