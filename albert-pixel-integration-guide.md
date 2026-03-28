# FeedMob Pixel 直接集成方案（无需 GTM）

## 概述

本文档说明如何在**不使用 Google Tag Manager (GTM)** 的情况下，将 FeedMob Pixel 直接集成到网站中。以 Albert 网站为 Demo 示例，演示完整的事件追踪流程。

---

## 架构对比

### 传统方式（使用 GTM）

```
用户操作 → GTM dataLayer.push() → GTM 容器 → 触发自定义 HTML 代码 → fmpix('event', 'REG')
```

### 直接集成方式（本方案）

```
用户操作 → JavaScript 直接调用 → fmpix('event', 'REG')
```

**优势：**
- 减少中间环节，事件触发更可靠
- 不依赖第三方容器加载
- 调试更直观，问题排查更简单
- 页面加载更快（无需加载 GTM 容器脚本）

---

## 集成步骤

### 第一步：在 `<head>` 中安装 Pixel 代码

将以下代码片段放到网站所有页面的 `</head>` 标签之前：

```html
<!-- Start Feedmob Pixel Snippet -->
<script>
!function(e,t,n,s,a,p,c){e[s]||((a=e[s]=function(){a.process?a.process.apply(a,arguments):a.queue.push(arguments)}).queue=[],a.t=+new Date,(p=t.createElement(n)).async=1,p.src="https://feedmob-cdn.s3.amazonaws.com/js/fmpixel.js?t="+864e5*Math.ceil(new Date/864e5),(c=t.getElementsByTagName(n)[0]).parentNode.insertBefore(p,c))}(window,document,"script","fmpix"),fmpix("init","{CLIENT_UUID}"),fmpix("event","pageload");
</script>
<!-- End Feedmob Pixel Snippet -->
```

**需要替换的参数：**
- `{CLIENT_UUID}` → 客户的唯一标识符（由 FeedMob 分配）

> 示例中的 UUID 为测试值：`d7ab0ac71e9245f48c8d92d59140bfb6`

**这段代码做了什么：**

| 动作 | 说明 |
|------|------|
| 创建 `fmpix` 队列 | 在主脚本加载前，先把所有调用存入队列 |
| 异步加载 `fmpixel.js` | 从 CDN 加载核心追踪脚本，不阻塞页面渲染 |
| `fmpix("init", UUID)` | 初始化，设置客户标识 |
| `fmpix("event", "pageload")` | 自动触发页面加载事件 |

---

### 第二步：在用户操作中触发事件

直接在 JavaScript 中调用 `fmpix()` 函数即可触发事件，无需通过 GTM。

#### 示例：注册事件（REG）

```javascript
// 用户点击注册按钮时
document.getElementById('signup-form').addEventListener('submit', function(e) {
  e.preventDefault();

  // 直接调用 fmpix 触发注册事件
  fmpix('event', 'REG');

  // 也可以附带额外数据
  fmpix('event', 'REG', JSON.stringify({
    name: '用户名',
    email: 'user@example.com'
  }));
});
```

#### 示例：安装/下载事件（I）

```javascript
document.getElementById('download-btn').addEventListener('click', function() {
  fmpix('event', 'I');
});
```

#### 示例：购买事件（P）

```javascript
function onPurchaseCompleted(amount) {
  fmpix('event', 'P', JSON.stringify({ amount: amount }));
}
```

---

## 支持的事件列表

| 事件代码 | 事件名称 | 触发场景 | 是否需要附加数据 |
|---------|---------|---------|---------------|
| `I` | first_install | 用户首次安装 App | 否 |
| `R` | retained | 用户回访 | 否 |
| `T` | tutorial | 完成新手引导 | 否 |
| `REG` | first_registration | 用户首次注册 | 可选 |
| `P` | first_purchase / all_purchase | 用户完成购买 | 推荐（amount） |
| `L` | level | 达到某个等级 | 否 |
| `O` | open | App 打开 | 否 |
| `M` | impression | 广告曝光 | 否 |
| `AA` | all_event_a | 自定义事件 A | 可选 |
| `FA` | first_event_a | 首次自定义事件 A | 可选 |
| `AB` | all_event_b | 自定义事件 B | 可选 |
| `FB` | first_event_b | 首次自定义事件 B | 可选 |

**自动触发的事件：**
- `pageload` — 页面加载时（由 snippet 自动触发）
- `pageclose` — 页面关闭/跳转时（由 Pixel 自动触发）

---

## Demo 页面事件流程

Demo 页面（`tmp/albert-demo.html`）模拟了 Albert 网站的核心交互，完整展示事件追踪流程：

```
用户访问页面
  │
  ├── 自动触发: pageload 事件（snippet 中自动触发）
  │
  ├── 点击 "Sign up" / "Get started"
  │     └── 打开注册弹窗
  │           └── 填写表单并提交
  │                 └── 触发: REG 事件（带用户名和邮箱数据）
  │
  ├── 点击 "Download the app"
  │     └── 触发: I 事件（安装事件）
  │
  └── 关闭/离开页面
        └── 自动触发: pageclose 事件
```

---

## MMP 追踪链接配置

通过 FeedMob 平台生成的追踪链接示例：

```
https://your-website.com/?utm_source=FeedMob_{VENDOR}&utm_medium=cpc&utm_campaign=Albert_Q1&fm_conversion_id={CONVERSION_ID}&fm_publisher_id={PUBLISHER_ID}&fm_click_id={CLICK_ID}
```

**必需参数：**
| 参数 | 说明 |
|------|------|
| `utm_source` | 必须以 `FeedMob` 开头 |
| `fm_publisher_id` | FeedMob 发布者 ID |
| `fm_conversion_id` | FeedMob 转化 ID |

**可选参数：**
`utm_medium`, `utm_term`, `utm_content`, `utm_campaign`, `utm_partner`, `fm_click_id`

---

## 验证方法

### 1. 打开浏览器 DevTools

打开 Demo 页面后按 `F12`，切换到 **Network** 标签。

### 2. 筛选 Pixel 请求

在过滤框输入 `feedmob` 或 `tracker`。

### 3. 查看请求参数

每次触发事件，可以看到一个发送到 `pixel-api.feedmob.biz/tracker` 的请求：

```
https://pixel-api.feedmob.biz/tracker
  ?id=d7ab0ac71e9245f48c8d92d59140bfb6   ← 客户 UUID
  &uid=1-cwq4oelu-in95g8xy               ← 用户唯一 ID
  &ev=REG                                 ← 事件名称
  &ed={"name":"John","email":"..."}       ← 事件数据
  &v=1                                    ← Pixel 版本
  &dl=http://localhost:8080/...            ← 当前页面 URL
  &rl=                                    ← 来源页面
  &ts=1464811823300                       ← 时间戳
  &utm_source=FeedMob_...                ← UTM 来源
  &fm_click_id=12345                      ← FeedMob 点击 ID
  &...
```

### 4. 使用内置 Debug 面板

Demo 页面底部有一个 **FeedMob Pixel Event Monitor** 面板，点击展开后可以实时查看所有事件的触发记录。

---

## 本地测试

```bash
cd /Users/jason/rails/openpixel

# 启动本地服务器
npx http-server -p 8080

# 访问 Demo 页面
# http://localhost:8080/tmp/albert-demo.html

# 带追踪参数访问（模拟 MMP 链接点击）
# http://localhost:8080/tmp/albert-demo.html?utm_source=FeedMob_Test&utm_medium=cpc&utm_campaign=Albert_Demo&fm_conversion_id=CONV123&fm_publisher_id=PUB456&fm_click_id=CLICK789
```

---

## 关键文件

| 文件 | 说明 |
|------|------|
| `tmp/albert-demo.html` | Albert Demo 页面（直接集成方式） |
| `tmp/gtm-test.html` | GTM 集成方式对比页面 |
| `tmp/test.full.html` | 完整功能测试页面 |
| `dist/fmpixel.js` | Pixel 核心脚本（未压缩） |
| `dist/fmpixel.min.js` | Pixel 核心脚本（压缩版） |
| `INSTALLATION.md` | 官方安装文档 |
