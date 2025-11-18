## :shield: APIGan - Swagger 未授权检测插件 (Burp Suite)

一款 Burp Suite 插件，用于被动发现和手动添加 Swagger/OpenAPI 文档，并自动对 API 接口进行未授权访问探测。集成在 Burp UI 中，实时显示调用结果，并可一键发送到 Repeater 模块。

![Mode](https://img.shields.io/badge/mode-Passive%20%7C%20Manual-blue)
![Platform](https://img.shields.io/badge/platform-Burp%20Suite-orange)
![Report](https://img.shields.io/badge/report-In--UI%20Table-success)
![Safe](https://img.shields.io/badge/scope-Configurable-yellow)

---

### :sparkles: 核心特性

* **被动发现 (Passive Discovery)**
    * 自动从 Burp 流量（Proxy, Target）中被动识别 `text/html`, `application/javascript` 等响应，基于响应体内容智能发现 API 文档。
    * 可配置 `Enable Passive Scan` 开关，在需要时切换为纯手动模式。

* **手动提交 (Manual Submission)**
    * 支持在 Burp 任意位置（如 Proxy 历史）右键，选择 `Extensions` -> `APIGan` -> `Send to APIGan`。
    * **自动处理编码：** 解决手动提交中文乱码问题。
    * **WAF 绕过兼容：** 即使是 `.../api-docs;js` 这样的绕过 URL，手动提交也能正确识别。

* **多版本兼容**
    * 兼容 Swagger 1.x, V2, OpenAPI V3.0+ 的 JSON 文档格式。

* **安全与高度可控**
    * **危险API过滤:** 可自定义 `Dangerous Keywords` (危险关键字)，命中 URL 的请求将自动跳过 (Status: Filtered)。
    * **请求方法控制:** 可自定义 `Allowed Methods` (允许的方法)，默认仅 `get,post`，防止执行高风险操作。
    * **请求超时:** 可自定义 `Timeout`（超时，秒），防止单个请求卡死。

* **深度集成 Burp UI**
    * **实时交互表格:** 所有 API 实时展示在表格中，包含状态（Called, Filtered, Error）、方法、URL、状态码等。
    * **即时查看:** 点击表格任意行，可在下方查看器中立即显示完整的**请求 (Request)** 和**响应 (Response)**。
    * **一键重放:** 在请求查看器中，支持 `Ctrl+R` 或右键将请求（可修改）发送到 **Repeater** 模块进行深度测试。

* **强大的请求配置**
    * **`Force original domain`:** 强制使用发现文档的域名，忽略文档内声明的 `host` 或 `servers`（用于测试环境与生产环境域名不一致的场景）。
    * **`Custom Headers`:** 支持添加全局 `Cookie`, `Authorization` 等自定义头部，用于测试需要认证的接口。
    * **`Specify PreFix URL`:** 支持为所有 API 路径添加统一的URL前缀（例如 `/api/v1`）。

---

### :camera: 界面概览

**(APIGan 主界面截图 - 展示设置面板、表格、请求/响应查看器)**
``
<img width="2560" height="1380" alt="0ee3c0d6-9035-4dd2-99c1-dfde6685b4d0" src="https://github.com/user-attachments/assets/187ef5c5-9e3e-451a-b6c6-734c53ad9b82" />

**(右键菜单截图 - 展示 "Send to APIGan")**
``
<img width="2560" height="1380" alt="6fc6fca2-055d-4890-a0fb-5eb8ad08c806" src="https://github.com/user-attachments/assets/d8e4b3f5-410d-4942-a6ed-8a34b7575359" />

---

### :package: 快速开始与安装

1.  下载 `APIGan.jar` 文件（您编译后的文件）。
2.  打开 Burp Suite -> `Extensions` (扩展) -> `Extender` 标签页。
3.  点击 `Add` (添加)，在 `Extension type` 中选择 `Java`。
4.  点击 `Select file...`，选择您下载的 `APIGan.jar` 文件。
5.  点击 `Next`，插件加载成功。
6.  您会看到一个新的 `APIGan` 标签页出现，同时右键菜单 `Extensions` 下出现 `APIGan` 选项。

---

### :gear: 使用说明

#### 1. 配置

* **`Enable Passive Scan` (推荐):** 保持勾选。
* **`Force original domain` (按需):** 当您在测试环境（如 `test.example.com`）发现 API 文档，但文档内的 `host` 指向生产环境（如 `api.example.com`）时，**必须勾选**此项，确保请求发往测试环境。
* **`Custom Headers` (关键):** 如果 API 需要认证，请在此处填入您的 `Cookie: ...` 或 `Authorization: Bearer ...`（每行一个），然后点击 `Save`。

#### 2. 自动扫描 (被动)

1.  保持 `Enable Passive Scan` 开启。
2.  正常浏览您的目标网站，使用网站功能。
3.  `HttpListener` 会自动监听所有流经 Burp 的流量。
4.  一旦在响应体中发现 API 文档，插件将**自动**开始解析和请求，并实时更新 `APIGan` 标签页中的表格。

#### 3. 手动扫描 (主动)

1.  在 Burp 的任何地方（如 `Proxy` -> `HTTP history`）找到一个您认为是 API 文档的请求（例如 `.../v2/api-docs` 或 `.../api-docs;js`）。
2.  右键点击该请求。
3.  选择 `Extensions` -> `APIGan` -> `Send to APIGan`。
4.  插件将立即解析此响应，并开始请求。

#### 4. 分析结果

1.  切换到 `APIGan` 标签页查看表格。
2.  **重点关注：** `Status` 为 `Called` (调用成功, 状态码 2xx) 或 `Error` (调用失败, 状态码 4xx/5xx) 的请求。
3.  点击您感兴趣的行（例如一个 `401 Unauthorized` 或 `400 Bad Request` 的请求）。
4.  在下方的 `Request` 查看器中检查请求包。
5.  按 `Ctrl+R`（或右键）将其发送到 `Repeater` 模块。
6.  在 `Repeater` 中修改请求（例如添加/修改参数、更换 `Cookie`），进行更深入的授权测试。
