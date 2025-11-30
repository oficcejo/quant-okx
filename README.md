# OKX 自动量化交易机器人

基于 Python Flask 开发的 OKX 交易所自动量化交易机器人，采用**前后端分离架构**，提供 Web 界面管理账户、执行策略、AI 策略生成及历史数据回测。
<img width="1907" height="883" alt="image" src="https://github.com/user-attachments/assets/24ba308f-3bc5-4d41-bc92-9a9ed8453af1" />
<img width="1867" height="822" alt="image" src="https://github.com/user-attachments/assets/47f4a045-911b-4c73-b774-7478c578633e" />
<img width="1793" height="881" alt="image" src="https://github.com/user-attachments/assets/1709c090-a39f-4a39-909e-9ca123e3f558" />
<img width="1781" height="765" alt="image" src="https://github.com/user-attachments/assets/e2260d70-2ebe-48f3-9929-8570b0c6d1bf" />

## 系统架构

本系统采用**前后端分离**设计，需要同时运行两个进程：

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Web 前端      │     │   策略调度器     │     │   策略执行器    │
│   (app.py)      │────▶│  (scheduler.py) │────▶│(strategy_runner)│
│   端口: 5002    │     │   后台常驻      │     │   子进程        │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        └───────────────────────┴───────────────────────┘
                                │
                        ┌───────▼───────┐
                        │   quant.db    │
                        │  SQLite数据库  │
                        └───────────────┘
```

- **app.py** - Web 前端服务，处理 API 请求，管理配置
- **scheduler.py** - 后台调度器，监控数据库，自动启动/停止策略进程
- **strategy_runner.py** - 策略执行器，由 scheduler 调用运行具体策略

## 功能特性

### 📊 仪表盘
- 实时账户权益、可用余额展示
- 持仓信息列表
- 权益曲线图表

### 📈 策略管理
- 加载 `strategy/` 目录下的策略文件
- 支持选择交易对、K线周期、杠杆倍率
- **启动/停止策略**
- **编辑策略代码**（在线代码编辑器）
- **删除策略**（含日志和交易记录）
- 查看策略详情、执行日志、交易记录

### 🤖 AI 策略生成
- 集成 OpenAI 兼容接口
- 自然语言描述自动生成可执行策略代码
- 预览、编辑、保存生成的策略

### 📉 策略回测
- **数据库模式**: 使用本地同步的历史数据（推荐）
- **实时获取模式**: 从 OKX API 实时获取数据
- 回测结果展示：总收益、胜率、最大回撤等

### 📁 历史数据管理
- 从 OKX API 同步 K 线数据到本地
- 支持多交易对、多周期
- 数据统计和清理

### ⚙️ 系统设置
- OKX API 配置
- AI 模型 API 配置
- 代理设置
- **一键初始化数据库**（清空数据保留结构）

### K线周期支持
| 周期 | 说明 | 策略执行间隔 |
|------|------|-------------|
| `1m` | 1 分钟 | 60 秒 |
| `5m` | 5 分钟 | 300 秒 |
| `15m` | 15 分钟 | 900 秒 |
| `1H` | 1 小时 | 3600 秒 |
| `4H` | 4 小时 | 14400 秒 |
| `1D` | 1 天 | 86400 秒 |

## 目录结构

```
quant-okx/
├── app.py                  # Web 前端服务 (Flask)
├── scheduler.py            # 策略调度器 (后台常驻)
├── strategy_runner.py      # 策略执行器 (由scheduler调用)
├── quant_engine/           # 核心引擎
│   ├── okx_client.py       # OKX API 客户端
│   ├── strategy_framework.py # 策略框架和全局函数
│   ├── backtest_engine.py  # 回测引擎
│   ├── market_data.py      # 历史数据管理
│   ├── db.py               # 数据库操作
│   └── config_loader.py    # 配置加载
├── strategy/               # 用户策略文件
│   ├── ma_crossover.py     # 双均线交叉策略
│   ├── tqqq.py             # 马丁格尔策略
│   ├── qqq.py              # 反弹策略
│   └── gdxu.py             # 动量网格策略
├── templates/              # HTML 模板
├── static/                 # CSS 和 JavaScript
├── 配置.txt                # 配置文件
├── quant.db                # SQLite 数据库
└── requirements.txt        # 依赖列表
```

## 安装部署

### 1. 环境准备

```bash
# 克隆项目
git clone <repository-url>
cd quant-okx

# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Windows:
.\venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

### 2. 配置 API

编辑 `配置.txt` 文件或在 Web 界面"系统设置"中配置：

```
# okx交易所配置
OKX_API_KEY=your_api_key
OKX_SECRET_KEY=your_secret_key
OKX_PASSPHRASE=your_passphrase
OKX_API_ENDPOINT=https://www.okx.com
PROXY_URL=                           # 可选，如 http://127.0.0.1:7890
# AI Configuration (支持 OpenAI 兼容的 API)
OPENAI_API_KEY=ak_###########
OPENAI_API_BASE_URL=https://########
OPENAI_MODEL=########
```

### 3. 运行服务（重要！）

**必须同时运行两个进程：**

#### 终端 1 - 启动 Web 前端
```bash
python app.py
```
输出：
```
* Running on http://0.0.0.0:5002
```

#### 终端 2 - 启动策略调度器
```bash
python scheduler.py
```
输出：
```
Starting Scheduler...
```

### 4. 访问界面

打开浏览器访问：`http://localhost:5002`

### Docker 部署（可选）

```bash
# 构建并运行
docker-compose up -d

# 查看日志
docker-compose logs -f
```

## 使用说明

### 1. 配置 API
首次运行请在"系统设置"页面配置：
- OKX API Key、Secret、Passphrase
- AI 模型 API（可选，用于 AI 策略生成）
- 代理设置（如需要）

### 2. 策略管理

#### 启动策略
1. 在"策略管理"页面选择策略文件
2. 输入交易对（如 `BTC-USDT` 现货 或 `BTC-USDT-SWAP` 合约）
3. 选择 K 线周期（决定策略执行间隔）
4. 设置杠杆倍率
5. 点击"启动策略"

> ⚠️ 确保 `scheduler.py` 正在运行，否则策略不会实际执行！

#### 编辑策略
- 点击策略列表中的 **编辑** 按钮
- 在弹出的代码编辑器中修改代码
- 点击 **保存修改**

#### 删除策略
- 点击 **删除** 按钮（运行中的策略无法删除）
- 确认后将删除策略文件及所有相关日志、交易记录

### 3. 同步历史数据
在"回测系统"页面的"历史数据管理"面板：
1. 选择交易对（如 BTC-USDT）
2. 选择 K 线周期
3. 选择日期范围
4. 点击"同步数据"

### 4. 策略回测
1. 选择策略文件
2. 选择交易对和 K 线周期
3. 选择回测模式：
   - **数据库模式**: 使用已同步的本地数据（推荐，速度快）
   - **实时获取模式**: 从 OKX API 实时获取数据
4. 设置初始资金
5. 点击"开始回测"

### 5. AI 策略生成
1. 在"AI 策略生成"页面输入策略描述
2. 点击生成
3. 预览生成的代码
4. 满意后保存到 strategy 目录

### 6. 数据库管理
在"系统设置"页面底部：
- 点击 **一键初始化数据库** 可清空所有日志、交易记录、K线数据
- 保留表结构，不影响策略文件

## 交易对说明

| 类型 | 格式 | 示例 | 交易模式 |
|------|------|------|----------|
| 现货 | `XXX-USDT` | `BTC-USDT` | cash |
| 永续合约 | `XXX-USDT-SWAP` | `BTC-USDT-SWAP` | cross |

> 系统会自动根据交易对类型选择正确的交易模式（tdMode）

## 内置策略说明

| 策略文件 | 策略类型 | 说明 |
|---------|---------|------|
| `ma_crossover.py` | 双均线交叉 | 经典趋势跟踪，金叉买入死叉卖出 |
| `tqqq.py` | 马丁格尔 | 递进式加仓，带止盈机制 |
| `qqq.py` | 反弹策略 | 逢跌买入，逢涨卖出 |
| `gdxu.py` | 动量网格 | 基于价格动量的仓位调整 |
| `grid_strategy.py` | 网格交易 | 固定区间网格策略 |
| `breakout_strategy.py` | 突破策略 | 价格突破买入/跌破卖出 |

## 策略开发指南

策略需继承 `StrategyBase` 类并实现以下方法：

```python
class Strategy(StrategyBase):
    def initialize(self):
        declare_strategy_type(AlgoStrategyType.SECURITY)
        self.symbol = declare_trig_symbol()
        # 初始化变量
        self.threshold = show_variable(0.02, GlobalType.FLOAT)

    def handle_data(self):
        # 主交易逻辑，每个周期执行一次
        crt_price = current_price(symbol=self.symbol, price_type=THType.FTH)
        if crt_price <= 0:
            return
        # ... 交易逻辑 ...
```

### 可用 API 函数

| 函数 | 说明 | 返回值 |
|------|------|--------|
| `current_price(symbol, price_type)` | 获取当前价格 | float |
| `max_qty_to_sell(symbol)` | 获取可卖数量（持仓量） | float |
| `max_qty_to_buy_on_cash(symbol, order_type, price)` | 获取可用 USDT | float |
| `position_pl_ratio(symbol, cost_price_model)` | 获取持仓盈亏比 | float |
| `place_limit(symbol, price, qty, side, time_in_force)` | 下限价单 | dict |

### 枚举类型
```python
# 订单方向
OrderSide.BUY / OrderSide.SELL

# 有效期
TimeInForce.GTC  # 撤销前有效
TimeInForce.IOC  # 立即成交或取消
TimeInForce.FOK  # 全部成交或取消

# 订单类型
OrdType.LMT  # 限价单
OrdType.MKT  # 市价单

# 价格类型
THType.FTH  # 最新成交价

# 策略类型
AlgoStrategyType.SECURITY
```

## 常见问题

### Q: 策略启动后显示 RUNNING 但没有执行？
A: 确保 `scheduler.py` 正在后台运行。Web 前端只负责更新数据库状态，实际策略由 scheduler 启动。

### Q: 下单失败 "All operations failed"？
A: 检查：
1. 网络是否能访问 OKX API（可配置代理）
2. API Key 是否正确且有交易权限
3. 账户余额是否充足

### Q: 合约下单报错 51121？
A: 合约数量必须为整数（张数），系统已自动处理转换。

### Q: 如何查看策略执行日志？
A: 点击策略列表中的"查看"按钮，可查看执行日志和交易记录。

## 注意事项

- ⚠️ **实盘交易存在风险，请务必先用小资金测试策略**
- 请确保网络能够访问 OKX API，如需代理请在配置中设置 `PROXY_URL`
- K 线周期决定策略执行间隔：选择 1H 则每小时执行一次
- 策略日志和交易记录保存在本地数据库 `quant.db` 中
- 建议根据交易对波动性选择合适的 K 线周期
- **必须同时运行 `app.py` 和 `scheduler.py` 才能正常使用**

## License

MIT License
