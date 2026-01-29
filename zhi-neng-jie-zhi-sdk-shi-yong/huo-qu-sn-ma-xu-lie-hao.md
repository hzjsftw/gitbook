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
