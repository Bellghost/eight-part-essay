# Playwright 八股文面试题

> 整理了高频 Playwright 面试题，附参考答案和解析，帮助你更好地准备面试。

---

## 🧠 基础概念篇：别被简单题淘汰

这类题目看似基础，但回答不好可能会让面试官质疑你的实战经验。

### Q1: Playwright 的优势是什么？

回答不能太宽泛，要答到点子上。

- **自动等待 (Auto-waiting)**：执行 `click`、`fill` 等操作前，**自动等待元素变为可交互状态**（可见、可用、稳定），大大减少了 `sleep` 或显式等待的使用。
- **真正的跨浏览器**：一套 API 可同时在 **Chromium、Firefox 和 WebKit** 上运行。
- **开箱即用**：内置并自动管理浏览器驱动，无需额外下载和配置。
- **强大的工具链**：内置网络拦截、视频录制、追踪查看器（Trace Viewer）等，调试体验极佳。

---

### Q2: Playwright 的劣势是什么？

了解工具的不足也很重要。

- **安装包较大**：每个浏览器约 250–300 MB。
- **生态系统较新**：2020 年发布，相比 2004 年的 Selenium，历史积累和第三方集成较少。
- **不支持旧浏览器**：设计上不支持 IE 和旧版 Edge。
- **学习曲线**：高级功能（如网络拦截、组件测试）有一定上手难度。

---

### Q3: Playwright 和 Selenium 的核心区别是什么？

不要只说"更快"，要说出核心的设计理念差异。

| 对比维度 | Playwright | Selenium |
|---------|-----------|---------|
| 等待机制 | **自动等待** | 需手动编写显式等待 |
| 跨浏览器 | 一套代码运行于 Chrome / Firefox / Safari | 需为不同浏览器配置不同驱动 |
| 内置功能 | 内置录屏、Mock、移动端模拟等 | 需依赖第三方插件 |

---

### Q4: 如何打开一个可见的浏览器窗口（非无头模式）？

```javascript
const browser = await chromium.launch({ headless: false });
```

通常在本地调试、录制操作或向他人演示时使用。

> **小技巧**：本地开发用 `headless: false`，在 CI（持续集成）环境中务必设为 `true` 以节省资源。

---

### Q5: `page.goto()` 会等待页面完全加载吗？

- 默认会等待页面触发 **`load` 事件**（即所有资源加载完毕）。
- 对于**动态加载数据**的页面，需要进一步等待：

```javascript
await page.goto('/product');
await page.waitForLoadState('networkidle'); // 等待网络空闲
// 或者更精准地等待某个元素出现
await page.waitForSelector('.product-list', { state: 'visible' });
```

> **注意**：`'networkidle'` 参数在 v1.25+ 已弃用，推荐使用 `waitForLoadState('networkidle')`。

---

### Q6: 如何处理 `alert` 弹窗？

必须在**触发弹窗之前**注册监听器。

```javascript
page.on('dialog', async dialog => {
    console.log('弹窗内容：', dialog.message());
    await dialog.accept();  // 点击"确定"
    // await dialog.dismiss(); // 点击"取消"
});
```

---

## ⚙️ 核心机制与工程实践篇

### Q7: 为什么推荐使用 `locator` 而不是 `$` 或 `$$`？

- `locator` 是 **Playwright 的核心推荐方式**，它具备自动等待和重试能力。
- `$` 和 `$$`（对应 `querySelector`）是**原生 DOM API**，执行一次即返回，不具备自动等待能力，容易因元素未加载而导致测试不稳定。

---

### Q8: 什么是"自动重试断言"？

Playwright 的断言（如 `expect(locator).toBeVisible()`）会**自动重试**，直到断言成功或超过超时时间（默认 5 秒）。这能有效应对动态页面，避免使用 `sleep`。

---

### Q9: 如何优化 Playwright 测试的运行速度？

可以从以下几个方面入手：

- **并行执行**：利用 Playwright 的并行测试能力。
- **无头模式**：在 CI 环境使用 `headless: true`。
- **复用登录态**：通过 `storageState` 保存和复用登录状态，避免每个测试都重复登录。
- **减少等待**：用精准的 `waitForSelector` 替代固定的 `page.waitForTimeout`。

---

## 🛠️ 常见问题与避坑篇

### Q10: 遇到"元素未找到"错误怎么办？

- **检查选择器**：在浏览器开发者工具中验证选择器是否正确。
- **增加等待**：使用 `page.waitForSelector(selector)` 等待元素出现。
- **检查是否在正确的框架中**：如果页面包含 `<iframe>`，需要先切换到对应的 `frame`。

---

### Q11: 如何解决测试不稳定（Flaky tests）的问题？

- **使用自动等待**：充分利用 Playwright 的自动等待机制。
- **隔离测试**：每个测试使用独立的 `browserContext`，保证状态干净。
- **使用 Mock 数据**：模拟网络请求，排除后端不稳定因素的影响。

---

## 💡 总结与建议

准备 Playwright 的"八股文"，关键在于**理解其设计哲学**，而不是死记硬背。

- **理解"为什么"**：面试官更想听到你对"自动等待"、"网络拦截"等特性**设计初衷**的理解。
- **结合实战**：将理论知识与你的项目经验结合，比如"我在项目中用 `page.route()` 解决了什么问题"。
- **亲手实践**：如果只用过 Selenium，花 10 分钟跑一遍 Playwright 官方示例，亲自感受其不同。
