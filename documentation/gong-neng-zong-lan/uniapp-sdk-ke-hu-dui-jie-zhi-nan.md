---
description: BCL 戒指Uni-app SDK 客户集成、使用及接口调用示例
icon: layers
---

# Uni-app SDK 客户对接指南

本文面向客户研发人员，说明 BCL 戒指 TypeScript SDK 的集成、蓝牙调用、历史数据和云服务接入。本文“原生小程序”特指微信原生 JavaScript 小程序；支付宝、抖音等平台不在本次交付范围。文档核对日期为 2026-11-17。

## 1. 交付内容与支持边界

| 项目 | 说明 |
| --- | --- |
| SDK 文件 | `[libs/bcl-ring-sdk](https://github.com/BravechipSpace/ChipletRing-APPSDK/tree/main/Uniapp/library)` |
| 文件格式 | ESM，使用 import |
| 示例工程 | `[uniapp-demo/](https://github.com/BravechipSpace/ChipletRing-APPSDK/tree/main/Uniapp)` |
| 蓝牙核心 | 扫描、连接、服务发现、Notify、协议指令、持续测量、历史同步、复合指令 |
| 网络模块 | 同一 bundle 导出 `BCLRingNetworkSDK`，需注入平台请求适配 |
| 验证范围 | 重点为真实 iOS / Android 手机中的微信小程序；编译、模拟测试不等于真机验收 |

SDK 文件由对接人员随版本交付。仓库中的 `@bcl/*` 包为 workspace 内部包，不应假定已发布到公共 npm。客户接入无需安装 Swift / Java 原生 SDK，也不需要在页面重新封包。HID、ECG、语音、文件系统等原生 SDK 能力，不能因为旧文档列出就认定本 SDK 已全部实现；请按实际交付接口确认。

本指南针对传统 uni-app Vue 3，不是 uni-app x / UTS。App 端基础 BLE 已有适配，但文件选择与 OTA Demo 重点实现于微信端；H5、其他小程序和鸿蒙不作为当前已验收范围。仓库同名 .d.ts 引用了 packages/ 声明，不能视为独立可分发类型包。

## 2. 集成与工程配置

1. 将本平台交付的 `bcl-ring-sdk.js` 放入客户工程的 `libs/`。
2. 在 HBuilderX 打开 Vue 3 工程，在 manifest.json 的 mp-weixin.appid 填写客户 AppID；运行到微信开发者工具后使用真机预览。SDK 是本地 ESM 文件，微信端无需再构建 npm。
3. 在隐私说明中声明蓝牙用途，按当前微信隐私与系统权限要求处理授权。手机系统蓝牙开关和微信蓝牙权限均需允许；Android 某些系统还需定位权限 / 定位开关。
4. 如使用云服务，配置第 5 节的 HTTPS 合法域名和 Token。
5. 使用真实手机和真实戒指验证，先关闭其他占用戒指的 App / 小程序。开发者工具模拟器不能替代 BLE 真机测试。

原生 App 发布时，在 `manifest.json` 的 `app-plus.modules` 启用 `Bluetooth`。Android 应按目标系统版本配置并申请蓝牙权限；仓库 Demo 声明包含 `BLUETOOTH`、`BLUETOOTH_ADMIN`、`BLUETOOTH_SCAN`、`BLUETOOTH_CONNECT`、`ACCESS_FINE_LOCATION`，声明本身不等于运行时已授权。iOS 按打包配置提供 `NSBluetoothAlwaysUsageDescription` 等隐私用途说明；使用相应旧系统时核对 `NSBluetoothPeripheralUsageDescription`。修改原生模块和权限后重新打包基座 / App 并在真实设备验收。

如从完整源码重新生成交付文件，在仓库根目录执行以下命令；仅收到单文件 SDK 的客户无需执行：

```bash
pnpm --filter @bcl/ring-uniapp-demo bundle:sdk
```

## 3. 扫描、连接与会话管理

### 3.1 最小业务封装

创建 `utils/bcl-ring.js`。该示例展示单设备会话，调用方串行执行扫描、连接和释放，并在按钮处理中处理异常。不要为每个页面分别创建 SDK 会话。

```js
import { scanUniAppRingDevices, connectUniAppRingSDK } from '@/libs/bcl-ring-sdk.js';

/** 当前设备会话；不放入页面响应式数据或 setData。 */
let connected;
/** 当前电量推送的取消订阅入口。 */
let unsubscribeBattery;

/** 扫描期间由 SDK 初始化蓝牙，结束后自动停止扫描。 */
export async function scanRings() {
  return scanUniAppRingDevices({
    uni, durationMs: 5000, filterBclManufacturerData: true,
    includeConnectedDevices: true,
  });
}

/** 由用户从扫描结果中选择设备，禁止默认连接列表第一项。 */
export async function connectRing(device) {
  if (!device || !device.deviceId) throw new Error('请先选择设备');
  if (device.mode === 'phyBootloader') throw new Error('请进入 PHY 固件恢复流程');
  await disconnectRing();
  connected = await connectUniAppRingSDK({
    uni, deviceId: device.deviceId, connectTimeoutMs: 10000,
    // uni adapter 不自行区分系统；iOS/未知平台不要传入期望 MTU。
    preferredMtu: uni.getSystemInfoSync().platform === 'android' ? 247 : undefined,
    writeType: 'writeNoResponse',
    command: {
      // 长测量按业务设置足够的响应等待时间，单位毫秒。
      timeoutMs: 60000,
      debug: (event) => {
        // 保留发送参数和原始 Hex，但须写入受控诊断通道。
        // 客户可在此调用自己的诊断模块，勿直接公开输出健康数据。
      },
    },
  });
  unsubscribeBattery = connected.sdk.onBatteryPush((event) => {
    // 在此更新业务电量状态，先检查 event.isLevelValid。
  });
  await connected.sdk.syncTime({
    // 示例使用中国时区。其他地区请传设备支持且项目确认的整数小时偏移。
    timestampMs: Date.now(), timezone: 8,
  });
  return connected;
}

/** 所有页面通过同一入口获取当前 SDK，断连后不复用旧实例。 */
export function getRingSDK() {
  if (!connected) throw new Error('请先连接戒指');
  return connected.sdk;
}

/** 停止监听并释放物理连接；失败应由业务提示并记录。 */
export async function disconnectRing() {
  const previous = connected;
  connected = undefined;
  if (unsubscribeBattery) unsubscribeBattery();
  unsubscribeBattery = undefined;
  if (previous) await previous.disconnect();
}

```

连接工厂会初始化蓝牙、建立连接、发现服务和特征、开启通知并创建 SDK；服务/通知初始化失败时会清理连接。不要在工厂返回前发送指令。时间同步若失败，会话仍可能已建立，调用方应提示重试或执行 `disconnectRing()` 清理。

### 3.2 页面接入与生命周期

在 Vue 页面中使用 `import { scanRings, connectRing, getRingSDK, disconnectRing } from "@/utils/bcl-ring.js"`；扫描按钮取得数组后更新页面设备列表，选择事件调用 `await connectRing(device)`。

按钮处理函数应设置处理中状态，防止重复扫描、重复连接及同时切换设备。若扫描 / 连接尚未结束时离页，等待操作结束后再释放；不要只在卸载时清理一次就忽略迟到连接结果。多页共享会话由应用级管理器统一拥有，页面只订阅自己需要的状态。

业务还需注册 `uni.onBLEConnectionStateChange(listener)`，只处理当前 deviceId 的真实断连，更新页面并关闭旧 SDK；卸载所有者时用对应 `offBLEConnectionStateChange(listener)` 注销同一函数（运行端需支持该取消监听接口）。不要注销其他模块的全部监听。断连后重新扫描 / 连接，不复用已 close 的 SDK。

普通业务可以按产品策略维持前台会话；不要承诺锁屏或后台持续测量。OTA 的后台取消和独占处理见第 7 节。

### 3.3 扫描和连接参数

| 参数 / 返回字段 | 说明 |
| --- | --- |
| `durationMs` | 扫描时长，默认 5000 毫秒；避免同时启动多次扫描 |
| `filterBclManufacturerData` | 按 BCL 厂商广播过滤；定制广播需与对接人员确认 |
| `namePrefix` / `nameIncludes` | 名称过滤，可选，不应硬编码其他客户型号 |
| `includeConnectedDevices` | 合并指定服务的系统已连接设备，不等同全部已配对列表；仍需建立 SDK 会话 |
| `includePhyBootloader` | 启用后允许 PHY Boot 候选通过 BCL 厂商广播过滤。名称、自定义等其他过滤仍生效 |
| `mode` / `applicationMac` | `mode` 可为 `application`、`phyBootloader` 或 `unknown`。PHY Boot 候选有可解析 MAC 时，SDK 会另外推导应用态 `applicationMac` |
| `onDeviceFound` | 调试回调包含未通过过滤的设备；业务选择列表优先使用最终返回的过滤结果 |
| `deviceId` | 平台连接标识；iOS 常是 UUID，不能假定为 MAC |
| `macAddress` | 符合厂商广播规则时解析；系统已连接设备可能没有此字段 |
| `connected.channel` | 已发现的实际设备 / 服务 / 特征 ID |
| `connected.currentMtu` | 主通道读取的实际 ATT MTU，可能为空；不代表 OTA 专用通道最终 MTU |
| `preferredMtu` | uni adapter 收到此值会尝试设置；业务仅在 Android 传入，iOS / 未知平台省略 |

普通默认 UUID 为 Service `0001`、Write `0010`、Notify `0011`。定制固件可通过 `serviceUuid`、`writeCharacteristicUuid`、`notifyCharacteristicUuid` 覆盖，以实际发现的 GATT 属性为准。

## 4. 常用蓝牙接口

以下 `sdk` 均为连接结果的 `connected.sdk`。除事件订阅外，方法返回 Promise；请在业务按钮处理函数中 `try/catch`。长任务按顺序执行，同一设备不要同时测量、同步历史和升级。

| 功能 | 调用方式 | 返回与边界 |
| --- | --- | --- |
| 同步时间 | `sdk.syncTime({ timestampMs: Date.now(), timezone: 8 })` | 时间戳用毫秒。时区使用有符号整数小时，半小时与四分之一小时时区无法精确表示 |
| 读取时间 | `sdk.getTime()` | `{ timestampMs, timezone }` |
| 固件 / 硬件版本 | `sdk.getFirmwareVersion()` / `sdk.getHardwareVersion()` | `{ version }` |
| 电量 | `sdk.getBatteryLevel()` | `{ level, isLevelValid, isCharging, isFull }`；先判断 `isLevelValid`，101/102 不应显示为百分比 |
| 充电状态 | `sdk.getChargingState()` | `state` 为 `not_charging` / `charging` / `full` |
| 实时步数 | `sdk.getStepCount()` | 返回 SDK 解析结果；累计步数不能逐条求和 |
| 采集周期 | `sdk.setCollectionPeriod({ periodSeconds: 300 })` / `sdk.getCollectionPeriod()` | 秒；0 关闭周期采集，非零最小 60；设置后检查 `success` |
| 蓝牙名称 | `sdk.setBluetoothName({ name: 'BCL-Ring' })` / `sdk.getBluetoothName()` | UTF-8 最多 12 字节；设置后检查 `success` |
| 一键自检 | `sdk.oneKeySelfInspection()` | `hasError`、`faults`、`description` |
| 温度 | `sdk.measureTemperature({ onProgress })` | 终态返回结果；温度用 `temperatureCelsius` 展示 |
| 电量推送 | `sdk.onBatteryPush(listener)` | 返回取消监听函数，事件含有效值标志、原始 Hex |
| 充电变化 | `sdk.onChargingStateChanged((state, event) => {})` | 返回取消监听函数 |
| 复合绑定 | `sdk.appEventBind({ request: { timezone: 8 } })` | 戒指侧绑定复合指令；不等同云端绑定 |
| 复合连接 / 刷新 | `sdk.appEventConnect(options)` / `sdk.appEventRefresh(options)` | 返回设备快照与历史；`request` 可指定 `timestampMs`、`timezone`、秒级 `filterTimestamp` |
| GoMore | `sdk.queryGoMoreAuthStatus()` / `sdk.readGoMoreSleepData(options)` | 需固件支持及对应算法授权；读取结果为 `models` |

`createUniAppRingSDK` 用于业务已经建立连接并准备好通道的高级场景，不会代替完整连接流程；普通集成优先使用 `connectUniAppRingSDK`。`sdk.close()` 关闭 SDK 请求和监听，**不会主动关闭物理 BLE 连接**；释放完整会话请调用 `connected.disconnect()`。

清步数、清历史、恢复出厂入口分别为 `clearStepCount()`、`deleteAllHistoryData()`、`restoreFactorySettings()`，只应在明确的用户操作中调用。不要在连接成功时自动执行。

### 4.1 主动测量示例

下面是异步业务函数内的调用片段。心率、血氧需分别执行，保持佩戴且设备处于可测量状态。

```js
// 持续回调更新页面；Promise 到设备终态才结束，不是“开始命令已发出”。
const heart = await sdk.startHeartRate({
  request: { collectTime: 30, collectFrequency: 50,
    waveformConfig: 0, progressConfig: 1, intervalConfig: 0 },
  onStatusChanged: (status) => console.info('心率状态', status),
  onProgress: (event) => console.info('心率进度', event.progress),
  onMeasureValue: (event) => {
    // 健康数值只更新授权页面，避免明文写入共享日志。
    renderHeartRate(event.heartRate);
  },
});
// heart 为最终事件；业务应以终态结果判断完成。
const oxygen = await sdk.startBloodOxygen({
  request: { collectTime: 30, collectFrequency: 25,
    waveformConfig: 0, progressConfig: 1 },
  onProgress: (event) => console.info('血氧进度', event.progress),
  onMeasureValue: (event) => renderBloodOxygen(event.bloodOxygen),
});
```

`renderHeartRate`、`renderBloodOxygen` 是业务页面提供的显示函数。用户停止操作应由另一个事件调用 `sdk.stopHeartRate()` 或 `sdk.stopBloodOxygen()`，并处理正在等待的测量 Promise 的拒绝。错误对象的 `status` 可用于区分未佩戴、忙碌、充电、超时、已停止等状态；并非所有设备都返回所有扩展字段。

### 4.2 历史同步与数据单位

```js
// 返回完整同步结果，不能只根据某一次进度回调判断成功。
const history = await sdk.syncUnuploadedHistoryData({
  timestamp: 0,
  onProgress: (total, sequence, progress) => {
    console.info('历史同步', { total, sequence, progress });
  },
  onMalformedRecord: (_error, rawHex) => {
    // 原始协议 Hex 仅进入受控诊断日志，可能包含健康数据。
    appendProtectedDiagnostic('历史异常包', rawHex);
  },
});
// history.records 是蓝牙模型；上传前需转换为云端模型（见第 6 节）。
renderHistory(history.records);
```

`renderHistory`、`appendProtectedDiagnostic` 由业务实现。全量同步调用 `syncAllHistoryData(options)`；停止调用 `stopHistoryDataUpload()`。`no_data` 是正常无数据终态。对异常记录应记录并复核，不能把缺包误当完整数据。蓝牙同步完成不代表云端上传成功，应用应先可靠保存数据，再处理上传与失败补偿；不要上传前自动清除戒指数据。

| 字段 / 接口 | 单位 / 含义 |
| --- | --- |
| `syncTime.timestampMs`、`getTime.timestampMs` | 毫秒 |
| 历史 `timestamp` / `timestampMs` | 秒 / 毫秒 |
| 历史 `temperatureRaw` / `temperatureCelsius` | 0.01 ℃ / ℃；云端 `temperature` 使用原始值 |
| 历史 `stepCount` | 当天累计步数；按时间和设备归并，不能直接累加每条记录 |
| 云端查询 `startTime`、`endTime` | 秒 |
| 云端时间线 `startTime`、`endTime`、主动测量 `testTime` 返回值 | 毫秒 |
| 历史心率、血氧、HRV、压力为 0 | 按无效值处理，不应直接参与平均计算 |

### 4.3 复合连接与刷新何时完成

本节的双顺序行为需要包含响应顺序修复的 SDK。源码已覆盖此修复，交付前请按第 2 节重新生成对应的 `bcl-ring-sdk.js`，并使用新文件验证。较早的 SDK 文件可能在先收到历史终态时一直等待，最后报告超时。

`appEventConnect()` 和 `appEventRefresh()` 成功时都需要设备信息和历史同步终态。两部分可按任意顺序到达。历史状态 `no_data` 表示设备没有可同步的记录，设备信息可能随后才到。请在 Promise 成功返回后更新整体完成状态。`result.finalEvent` 仍是历史终态事件，`result.records` 保存本次历史记录。设备负响应或历史失败会拒绝 Promise，缺少任一成功条件仍可能超时。

```js
// 在已有 BLE 会话中刷新。示例使用中国时区，0 表示不按历史时间过滤。
const result = await sdk.appEventRefresh({
  request: { timezone: 8, filterTimestamp: 0 },
});
// 此时设备信息和历史同步结果均已收到。
const { deviceInfo, records } = result;
```

## 5. 云端 SDK 集成

蓝牙与云端是两个独立模块：蓝牙连接不需要云端 Token，连接成功也不会自动建立云端绑定或上传记录。使用云服务前向对接人员申请服务权限与公司 Key，并确认用户唯一标识、地区和目标固件类别。

### 5.1 请求适配与 Token

SDK 不依赖页面登录流程，但需要 `fetch` 兼容适配。小程序和 uni-app 不应假定存在浏览器全局 fetch。将下列代码保存在 `utils/bcl-network.js`，与蓝牙模块使用同一平台的 SDK 文件。

```js
import { BCLRingNetworkSDK } from '@/libs/bcl-ring-sdk.js';

/** 递归隐藏授权信息、身份和健康数据，仅保留可排查请求结构的字段。 */
function maskLog(value) {
  if (typeof value === 'string') {
    try { return maskLog(JSON.parse(value)); } catch (_) { return '[文本已隐藏]'; }
  }
  if (Array.isArray(value)) return value.map(maskLog);
  if (value instanceof ArrayBuffer || ArrayBuffer.isView(value)) return '[二进制]';
  if (value && typeof value === 'object') {
    return Object.fromEntries(Object.entries(value).map(([key, item]) => [key,
      /auth|token|key|cookie|secret|user|phone|mac|name|data|value|heart|blood|temperature|sleep|wave|stress|rr|latitude|longitude/i.test(key)
        ? '***' : maskLog(item)]));
  }
  return value;
}

/** 仅实现当前网络 SDK 使用的 fetch 子集；保留字符串及 multipart 二进制正文。 */
function platformFetch(url, options = {}) {
  const method = options.method || 'GET';
  // 查询值全部隐藏，避免带签名 URL 或用户标识进入日志。
  const safeURL = String(url).split('?')[0];
  console.info('网络请求', method, safeURL,
    maskLog(options.headers), maskLog(options.body));
  return new Promise((resolve, reject) => {
    uni.request({
      url, method, header: options.headers || {}, data: options.body,
      dataType: 'text', responseType: 'text', timeout: 15000,
      success(response) {
        console.info('网络响应', method, safeURL, response.statusCode, maskLog(response.data));
        resolve({
          ok: response.statusCode >= 200 && response.statusCode < 300,
          status: response.statusCode,
          json: async () => typeof response.data === 'string'
            ? JSON.parse(response.data) : response.data,
        });
      },
      fail(error) {
        // 失败原因先移除 URL，再隐藏其余自由文本，完整诊断交给受控日志处理。
        const reason = String(error.errMsg || '').replace(/https?:\/\/[^\s]+/g, '[URL]');
        console.error('网络失败', method, safeURL, maskLog(reason));
        reject(new Error(reason || '网络请求失败'));
      },
    });
  });
}

/** 持久化当前 BCL Token；不与其他业务登录凭证混用。 */
const tokenStore = {
  getToken() {
    const value = uni.getStorageSync('bcl_ring_network_token');
    return typeof value === 'string' && value ? value : undefined;
  },
  setToken(token) {
    if (token) uni.setStorageSync('bcl_ring_network_token', token);
    else uni.removeStorageSync('bcl_ring_network_token');
  },
};
/** 国内默认基址，SDK 会自动添加 /api；baseURL 不要重复填写 /api。 */
export const networkSDK = new BCLRingNetworkSDK({
  region: 'domestic', fetch: platformFetch, tokenStore,
});

```

该适配器只用于 SDK 当前的请求方式，不是通用 fetch polyfill。客户工程可将 `console` 接到统一日志页，保留方法、URL、脱敏 Header/Body、HTTP 状态、响应结构与失败类别；详细健康数据及原始 Hex 只在授权的受控诊断流程保存。

在业务异步函数中完成登录态设置：

```js
// loginToken 必须是 BCL 云服务认可的 Token，不是任意业务系统的登录凭证。
await networkSDK.setToken(loginToken);
await networkSDK.refreshToken();
// 退出登录或切换账户时清除，避免后续接口沿用旧身份。
await networkSDK.clearToken();
```

需要 SDK 申请 Token 时调用 `await networkSDK.createToken(companyApiKey, stableUserId)`，成功后自动保存。生产环境建议公司 Key 留在客户后端，由后端换取 Token，再交给客户端 `setToken`；不要在示例、日志和公开代码中写入真实 Key。`refreshToken()` 仅延长服务端时效，不替换本地 Token。

### 5.2 区域与域名

- 国内默认 `https://ali.bravechip.cn`；例如绑定请求最终为 `/api/app/bindDevice`。
- 小程序后台将 API 域名加入 `request 合法域名`，固件实际下载域名加入 `downloadFile 合法域名`，含实际使用的重定向域名。开发工具跳过校验不代表正式版可用。
- 当前代码中的海外默认地址为 HTTP IP，不能直接作为微信正式版合法域名。海外服务需对接可用 HTTPS 域名或客户 HTTPS 代理；`region: 'overseas'` 不自动增加 `/api`，自定义 `baseURL` 时确认代理路径规则。
- 默认请求头含 `clientType: WeChatMP`、本地时区小时数 `timeZone`；Token 存在时自动添加 `Authorization: Bearer ...` 和 `access_token`。uni-app 发布原生 App 时请与服务方确认并配置 `clientType`。

## 6. 设备绑定、历史上传与查询示例

以下片段运行于业务异步函数内，`networkSDK` 来自上一节，`sdk` 和 `selectedDevice` 来自成功连接的当前会话。

```js
// 使用广播解析的真实 MAC；平台 deviceId（尤其 iOS UUID）不能充当 MAC。
const mac = selectedDevice.macAddress;
if (!mac) throw new Error('未获得真实 MAC，请重新扫描并核对广播数据');
await networkSDK.bindDevice(mac);
// 云端绑定与戒指侧 appEventBind 是两项操作，业务分别记录结果并处理失败。
const firmware = await sdk.getFirmwareVersion();
await networkSDK.updateDeviceInfo({ macAddress: mac, softwareVersion: firmware.version });

// 同步结果先持久化；只在成功后上传，空数据不调用上传接口。
const history = await sdk.syncUnuploadedHistoryData();
const uploadRecords = mapHistoryRecords(history.records, mac);
if (uploadRecords.length) await networkSDK.uploadHistory(uploadRecords, mac);

// 云端查询统一传秒，返回值由 SDK 去除服务端外层响应包装。
const endTime = Math.floor(Date.now() / 1000);
const startTime = endTime - 86400;
const records = await networkSDK.queryHistoryData({ macAddress: mac, startTime, endTime });
const sleep = await networkSDK.getSleepData({ startTime, endTime });
const timeline = await networkSDK.getTimeline(startTime, endTime);
```

蓝牙模型和云端字段不完全同名，不能直接 `uploadHistory(history.records, mac)`。下面转换覆盖基础字段、协议数组和已解析扩展，与 Demo 保持一致；函数放在上述业务模块中。

```js
/** 将蓝牙记录转换为云端模型，保留原始单位与扩展字段。 */
function mapHistoryRecords(records, mac) {
  return records.map((record) => mapHistoryRecordForUpload(record, mac));
}
/** 转换单条历史记录；缺失扩展以 null 提交。 */
function mapHistoryRecordForUpload(record, mac) {
  const extensions = record.extensions || {};
  return {
    id: null,
    mac: mac || null,
    totalNumber: record.totalNumber,
    indexNumber: record.sequence,
    time: record.timestamp,
    stepCount: record.stepCount,
    stepCountingType: 0,
    heartRate: record.heartRate,
    bloodOxygen: record.bloodOxygen,
    heartRateVariability: record.heartRateVariability,
    stressIndex: record.stressIndex,
    temperature: record.temperatureRaw,
    exerciseIntensity: record.activityIntensity,
    sleepType: record.sleepType,
    measurementMarker: record.piReserved,
    reserve: record.resReserved,
    rrCount: record.arrayCount,
    rrBytes: uint16ValuesToSignedBytes(record.arrayValues || []),
    measureType: extensions.measureType ?? null,
    stepsPerUnitTime: extensions.periodStepCount ?? null,
    accumulationFlag: extensions.runningFlag ?? null,
    batteryLevel: extensions.batteryLevel ?? null,
    batteryVoltage: extensions.batteryVoltageRaw ?? null,
    notWornReason: extensions.notWearingReason ?? null,
    recordCalibrationFlag: extensions.calibrationFlag ?? null,
    ambientLightIntensityFactor: extensions.ambientLight ?? null,
    temperatureData: getHistoryTemperatureData(record),
    stepRateMax: extensions.maxStepFrequency ?? null,
    stepRateMin: extensions.minStepFrequency ?? null,
    breathingRate: extensions.respiratoryRate ?? null,
    breathingRateConfidence: extensions.respiratoryConfidence ?? null,
    activityStatus: extensions.motionState ?? null,
    totalCalories: extensions.totalCalories ?? null,
    basalMetabolicCalories: extensions.basalCalories ?? null,
    exerciseCalories: extensions.activeCalories ?? null,
    hardAdt: extensions.hardAdt ?? null,
    highBloodPressure: extensions.systolicPressure ?? null,
    lowBloodPressure: extensions.diastolicPressure ?? null,
  };
}

/** 将协议 uint16 数组还原为小端有符号字节数组。 */
function uint16ValuesToSignedBytes(values) {
  const bytes = [];
  values.forEach((value) => {
    bytes.push(toSignedByte(value & 0xff));
    bytes.push(toSignedByte((value >> 8) & 0xff));
  });
  return bytes;
}

/** 云端要求的有符号字节数值。 */
function toSignedByte(value) {
  return value > 127 ? value - 256 : value;
}

/** 保留温度原始单位，按原生云接口序列化。 */
function getHistoryTemperatureData(record) {
  const temperatureField = (record.extensionFields || []).find((field) => field.type === 0x09);
  if (temperatureField && temperatureField.data && temperatureField.data.length) {
    return `[${parseTemperatureRawValues(temperatureField.data).join(",")}]`;
  }
  if (record.extensions && record.extensions.temperatures && record.extensions.temperatures.length) {
    return `[${record.extensions.temperatures.map((value) => Math.round(value * 100)).join(",")}]`;
  }
  return null;
}

/** 解码温度扩展中的有符号小端 16 位原始温度。 */
function parseTemperatureRawValues(data) {
  const count = data[0] || 0;
  const values = [];
  for (let index = 0; index < count; index += 1) {
    const offset = 1 + index * 2;
    if (offset + 1 >= data.length) {
      break;
    }
    const raw = data[offset] | (data[offset + 1] << 8);
    values.push(raw >= 0x8000 ? raw - 0x10000 : raw);
  }
  return values;
}

```

上传失败应保留本地记录并支持补传；对重复同步按设备、时间、序号等业务键去重。是否需要扩展字段、服务器去重和数据保留周期，应在项目联调时确认。

### 6.1 其他常用云端接口

| 场景 | SDK 调用 | 返回 / 参数说明 |
| --- | --- | --- |
| 最新记录 | `loadUserLatestHistory()` | 最新历史对象或 `null` |
| 保存主动测量 | `insertActiveMeasureData({ mac, testType: '心率', value: '72' })` | 测量成功后由业务主动调用；不会自动上传 |
| 查询主动测量 | `queryActiveMeasureData()` | 记录数组 |
| GoMore 睡眠上传 | `uploadGoMoreSleepData(models)` | 蓝牙 `readGoMoreSleepData()` 返回的 `models`；需实际算法支持 |
| GoMore 查询 | `getGoMoreSleepData({ startTime, endTime })` | 睡眠对象或 `null` |
| 周/月睡眠 | `getSleepDataByWeekOrMonth(['2026-11-01'])` / `getGoMoreSleepDataBatch(dates)` | 日期 `yyyy-MM-dd` |
| 运动汇总 | `getSportData({ startTime, endTime })` | 对象或 `null` |
| 查询设备 | `getDeviceByMac(mac)` | 对象或 `null` |
| 解绑关系 | `unbindDevice(mac)` | 不自动断开 BLE；先停止任务并释放会话 |
| 检查固件 | `checkFirmwareUpdate(version)` | `{ hasNewVersion, version, downloadUrl, fileName, ... }`；无新版本也是正常结果 |
| 历史固件 | `getFirmwareVersionList(category)` | `{ fileName, filePath, fileUrl }[]`；下载用 `fileUrl`，不是 `filePath` |
| 下载固件 | `downloadFirmware(url, fileName, destinationPath, onProgress)` | 需构造时注入 `downloader`；进度 0～1，成功返回平台路径 |
| 血压 / 血糖算法 | `uploadBloodPressureData(mac, waveData)` / `uploadBloodGlucoseData(mac, waveData)` | 需真实波形、算法权限及 gzip+base64 压缩器；不代表客户端可离线计算 |
| 日志文件上传 | `uploadFile(arrayBuffer, fileName, mimeType)` | 返回 `{ url, fileName }`；不是传电脑路径 |
| 问题反馈 | `questionFeedback(content, logPath, feedbackType, receivedFollow)` | `logPath` 通常用上传结果 `fileName` |

小程序运行时可能没有 `CompressionStream`，波形算法需要构造时注入 `compressWaveData(text)`，返回 gzip 后的 base64；不支持时抛出 `unsupported_compressor`。不要以普通 base64 替代 gzip。固件下载器和压缩器属于平台集成职责，不会因为导入 SDK 自动就绪。

云端服务业务说明继续参阅[获取 Token](../sheng-ji-fu-wu/huo-qu-token.md)、[睡眠服务](../sheng-ji-fu-wu/shui-mian-fu-wu.md)、[固件列表](../sheng-ji-fu-wu/huo-qu-gu-jian-lie-biao.md)；其中 Swift/Java 方法签名不适用于本 TypeScript SDK，客户端调用以本文为准。

## 7. 固件升级接入

先读取设备固件版本并查询更新，由客户明确选择适配固件，再读取二进制并校验；文件选择不触发升级。OTA 使用独立服务及原始传输通道，普通蓝牙命令通道不能直接当升级通道。

| 类型 | SDK 入口 | 文件、通道与当前范围 |
| --- | --- | --- |
| Apollo | `sdk.startApolloUpgrade(transport, firmwareBytes, options)` / `sdk.stopApolloUpgrade()` | `.bin`，独立 Apollo GATT；237 字节负载，实际 ATT MTU 至少 240 |
| Nordic | `sdk.startNordicUpgrade(transport, firmware, options)` / `sdk.stopNordicUpgrade()` | Secure DFU 单 application ZIP；需先解析 ZIP 为 NordicFirmware；标准 03/04 入口，不支持 Legacy、多镜像 |
| PHY | `sdk.startPhyUpgrade(transport, hexBytes, options)` / `sdk.stopPhyUpgrade()` | HEX16/HEX，独立 Boot 三特征，237 字节负载，实际 ATT MTU 至少 240；失败从首分区重刷 |

`transport` 不是扫描结果，也不是文件路径。它必须由对应 OTA 通道构建并提供通知与断连事件。Nordic 的 ZIP 解析、切换 Boot、目标关联与重连，PHY 的持久恢复记录、MAC 关联和应用态恢复还需要平台控制器；仅调用核心升级方法不包含全部业务流程。

首次接入建议完整保留交付 Demo 的升级页面 / 面板和辅助模块，跑通后再移植 UI：

- Uni-app：保留 `components/ApolloOtaPanel.vue`、`components/NordicOtaPanel.vue`、`components/PhyOtaPanel.vue` 及 `utils/apollo-ota.ts`、`utils/nordic-ota.ts`、`utils/phy-ota.ts`、`utils/wechat-platform.ts` 和实际引用组件、页面、libs、配置。聊天文件、文件系统与解压依赖微信 API；移植 App 时需替换这些平台实现，不能只替换蓝牙对象。


- Android 可请求 MTU 后读取实际结果；iOS 由系统管理，不主动设置。MTU 不足或未知时阻止依赖大包的升级，不能将期望值当作真实值。
- 升级时独占设备，禁止同时扫描换设备、发送普通命令或同步历史；停止、离页、后台事件需转交控制器处理，文件选择引发的暂时隐藏与实际传输后台取消要区分。
- 正文进度 100% 不代表校验、重启确认或应用态恢复完成。重连后展示真实固件版本，当前 Demo 不按文件名做目标版本比较。
- 取消无法撤销已提交写入，也不代表设备回滚。PHY 恢复需要真实广播 MAC；不要对 iOS UUID 做 MAC 运算，不按最强信号猜测同名 Boot 设备。

### 7.1 PHY Boot 恢复

在恢复入口扫描时传入 `includePhyBootloader: true`。将 `mode === 'phyBootloader'` 的候选送到 PHY 恢复页面。正常升级先连接应用态戒指，由 Demo 控制器切到 Boot 并保存恢复所需的固件副本及设备身份。已经在 Boot 的戒指可直接进入恢复流程。

恢复前确认 HEX16 / HEX 与目标硬件匹配。控制器重新扫描并核对目标广播 MAC、Boot 服务及特征，确认实际 ATT MTU 至少为 240 后才开始写入。找不到真实 MAC、目标不唯一或连接条件不足时，请停止并检查设备和日志。恢复会从首分区重刷；小程序重启后可使用已保存的固件和恢复记录重新搜索，固件副本缺失时需重新选择。

如果协议已完成但未成功恢复应用态，请使用 Demo 的“重新搜索并读取版本（不刷写）”入口。该入口只确认设备当前模式并读取实际版本。仍处于 Boot 时，按恢复流程重新选择目标。PHY 刷写及中断恢复仍需在项目使用的真机和固件上验收。

## 8. 错误处理

| 表现 | 排查与处理 |
| --- | --- |
| SDK 文件加载失败 | 检查平台文件格式、相对路径及交付完整性；两平台同名文件不可互换 |
| BLE 10000 / 10001 | 蓝牙未初始化 / 不可用；检查系统蓝牙、应用权限与实际运行环境 |
| BLE 10003 / 10006 / 10012 | 连接失败 / 断连 / 超时；释放旧会话，检查设备占用后重新扫描连接 |
| BLE 10004 / 10005 / 10007 | 服务 / 特征缺失或属性不支持；核对 UUID、固件模式及实际 GATT |
| Android 扫不到设备 | 核对系统版本对应的蓝牙、附近设备与定位授权及系统开关；记录原始广播，检查过滤条件 |
| 测量失败 | 按 `error.status` 提示佩戴、充电、忙碌、停止或超时，不只显示“调用失败” |
| `token_error` 或 HTTP 401 | 清除旧 Token，完成重新登录 / 换取 BCL Token，避免无限自动重试 |
| `http_error` / `server_error` | 分别检查 HTTP 状态和服务端业务状态；二者含义不同 |
| `invalid_parameters` / `invalid_response` / `decoding_error` | 检查参数和单位、服务响应字段与代理返回内容 |
| 请求 / 下载在正式版失败 | 核对 HTTPS、request/downloadFile 合法域名及重定向，不能依赖开发工具跳过校验 |
| `unsupported_compressor` / 未配置下载器 | 注入适配当前运行端的 gzip+base64 压缩器 / 固件下载器 |
| OTA 不能开始 | 检查固件、真实 MTU、设备身份和其他任务是否结束；查看控制器具体阻止原因 |
