# wechat_get
一个微信公众号自动化采集的工具
# 微信公众号文章抓取工具

使用 mitmproxy 拦截微信 PC 端公众号文章并同步到飞书多维表格。

**注意**: 本分支仅支持 Windows 平台。

## 功能特点

- **桌面自动化**: 自动控制微信 PC 客户端进行搜索
- **流量拦截**: 使用 mitmproxy 拦截文章 HTML
- **数据清洗**: 提取文章标题、作者、正文、发布时间等
- **数据同步**: 清洗后的数据同步到飞书多维表格

## 项目结构

```
weixin_get/
├── main.py                 # 主程序入口
├── pyproject.toml          # 项目配置
├── .env.example            # 环境变量示例
├── config/
│   └── settings.py         # 配置管理
└── src/
    ├── feishu/
    │   └── client.py       # 飞书 API 客户端
    ├── proxy/
    │   ├── interceptor.py  # mitmproxy 拦截器
    │   └── manager.py      # 代理管理器
    ├── parser/
    │   └── cleaner.py      # HTML 数据清洗
    └── automation/
        └── wechat_win.py   # Windows 桌面自动化
```

## 安装

### 前置条件

- Windows 操作系统
- Python 3.12+
- [uv](https://github.com/astral-sh/uv) (推荐) 或 pip
- 微信 PC 客户端

### 安装步骤

```bash
# 进入项目目录
cd /path/to/weixin_get

# 使用 uv 安装依赖
uv sync

# 复制环境变量配置文件
copy .env.example .env
```

### 配置

编辑 `.env` 文件配置飞书凭证（可选，用于同步到飞书）：

```env
# 飞书配置
FEISHU_APP_ID=your_app_id
FEISHU_APP_SECRET=your_app_secret
FEISHU_APP_TOKEN=your_bitable_app_token
FEISHU_TABLE_ID=your_table_id

# 代理配置
PROXY_HOST=127.0.0.1
PROXY_PORT=8080
```

## 使用方法

### 交互式模式（推荐）

```bash
uv run python main.py
```

按提示操作：
1. 选择运行模式（自动/手动）
2. 选择关键词来源（手动输入/飞书/文件）
3. 等待自动执行

### 命令行模式

```bash
# 指定关键词
uv run python main.py -k "人工智能" "机器学习"

# 从文件读取关键词
uv run python main.py -f keywords.txt

# 从飞书获取关键词
uv run python main.py --feishu

# 限制每个关键词的文章数
uv run python main.py -k "AI" -n 10

# 仅启动代理（手动操作微信）
uv run python main.py --proxy-only
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `-k, --keywords` | 直接指定关键词列表 |
| `-f, --file` | 从文件读取关键词 |
| `--feishu` | 从飞书多维表格获取关键词 |
| `-i, --interactive` | 交互式模式 |
| `-n, --max-articles` | 每个关键词最多抓取的文章数（默认5） |
| `--proxy-only` | 仅启动代理服务器模式 |

## 工作流程

```
1. 启动程序
     ↓
2. 初始化组件（飞书客户端、代理、自动化）
     ↓
3. 获取关键词（手动/飞书/文件）
     ↓
4. 启动 mitmproxy 代理 + 配置系统代理
     ↓
5. 自动化操作微信搜索关键词
     ↓
6. 点击文章 → 代理拦截 HTML
     ↓
7. 清洗数据 → 提取结构化信息
     ↓
8. 同步到飞书多维表格
     ↓
9. 处理下一个关键词...
```

## 飞书多维表格字段

需要在飞书多维表格中创建以下字段：

| 字段名 | 说明 |
|--------|------|
| 标题 | 文章标题 |
| 作者 | 文章作者 |
| 公众号 | 公众号名称 |
| 发布时间 | 文章发布时间 |
| 内容摘要 | 内容摘要（前200字） |
| 原文链接 | 原文 URL |
| 封面图 | 封面图 URL |
| 搜索关键词 | 搜索用的关键词 |
| 抓取时间 | 数据抓取时间 |
| 阅读量 | 文章阅读量 |

## 注意事项

1. **证书安装**: 需要安装 mitmproxy 的 CA 证书才能拦截 HTTPS 流量，将证书导入到「受信任的根证书颁发机构」
2. **微信版本**: 不同版本的微信 UI 可能不同，如需校准坐标请运行 `python calibrate.py`
3. **请求频率**: 建议适当控制频率，避免被限制

## 常见问题

### Q: 代理启动失败？
A: 检查端口 8080 是否被占用，可在 `.env` 中修改 `PROXY_PORT`

### Q: 文章无法拦截？
A: 需要安装 mitmproxy 证书，运行 `mitmproxy` 命令后访问 `mitm.it` 安装证书

## License

MIT

