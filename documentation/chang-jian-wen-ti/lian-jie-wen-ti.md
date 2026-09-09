---
description: 戒指出现了连接不上的问题，如何排查？
icon: plug
---

# 连接问题

### step 1

如果出现了戒指连接不上，第一步可以先放到充电仓里，充一会电，再重新连接，如果可以连接上，可能是戒指没电了

### step2

如果充完电，可以使用推荐的`nRF Connect`  软件尝试搜索一下戒指，看看能不能搜索到，能不能正常连接

### step3

如果是HID的戒指，可以通过系统蓝牙直接连接的，看看配对是否正常，如果不能配对，或者报错，那就是戒指不兼容或者坏了

### 戒指连接思路

戒指是连接在手机蓝牙上，所以app需要监听连接状态，这个sdk里已经封装好，建议在BaseActivity里设置监听。

除非特殊情况，戒指不需要一直长连接，因为数据是保存在戒指里，只需要在用户使用app的时候保持连接，同步历史数据即可，Android可以设置一下前台服务，这样可以保证，在app存活期间，戒指断连概率降低，app退到后台以后，为了节省功耗，可以定时5分钟断掉蓝牙，app到前台的时候，重连戒指，同步数据即可。

重连机制，可以参考公版app的重连

{% content-ref url="../zhi-neng-jie-zhi-sdk-shi-yong/images-and-media.md" %}
[images-and-media.md](../zhi-neng-jie-zhi-sdk-shi-yong/images-and-media.md)
{% endcontent-ref %}

{% content-ref url="../zhi-neng-jie-zhi-sdk-shi-yong/markdown.md" %}
[markdown.md](../zhi-neng-jie-zhi-sdk-shi-yong/markdown.md)
{% endcontent-ref %}

