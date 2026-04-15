# Exchange 项目结构分析报告

> 分析日期：2026-04-09
> 分析范围：`~/code2/exchange/` 目录下全部子项目

---

## 一、整体概述

`exchange` 是一个完整的加密/金融交易所技术平台，涵盖 **现货交易（OTC/Exchange）**、**合约/期货交易**、**银行类金融业务** 等多个业务线。项目包含 5 个顶级子项目：

| 子项目 | 类型 | 技术栈 | 定位 |
|---|---|---|---|
| `exchangeAndroid` | Android 移动端 | Kotlin / Android SDK | C端用户 App（现货 + 合约） |
| `exchangeContract` | Vue.js Web 管理端 | Vue2 + CoreUI Pro | 合约/期货管理后台 |
| `exchangeUi-bank` | Vue.js Web 管理端 | Vue2 + CoreUI Pro | 银行类 UI 组件库/模板 |
| `exchangeweb-bank` | Node.js 后端 + Vue 前端 | Egg.js + Vue2 | 银行 Web 平台后端服务 |
| `futures` | Java 后端（微服务） | Spring Boot + Maven | 期货/合约交易核心微服务 |

> **注意**：`exchangeContract` 中的 "Contract" 指的是 **合约/期货交易业务**，而非区块链 Solidity 智能合约。整个项目中未发现 `.sol` 智能合约文件或 Truffle/Hardhat 配置。

---

## 二、各子项目详细分析

---

### 2.1 exchangeAndroid（Android 客户端）

#### 技术栈

| 类别 | 技术 |
|---|---|
| 语言 | Kotlin（主）+ Java |
| 最低 SDK | 28（Android 9） |
| 目标 SDK | 34（Android 14） |
| UI 框架 | Android 原生（XML 布局） |
| 架构 | MVP（Activity/Fragment 作为 View 层） |
| 路由 | ARouter |
| 网络 | Retrofit + RxJava + OkHttp |
| 依赖注入 | Dagger2 |
| 图片加载 | Fresco |
| WebSocket | OkHttp WebSocket |
| 本地存储 | MMKV |
| 混淆/加固 | ProGuard |
| 编译 | Gradle 7.3.0 / Kotlin 1.7.10 |

#### 目录结构

```
exchangeAndroid/
├── app/                          # 主 App Module
│   ├── build.gradle
│   └── src/main/java/com/br/nk/
│       ├── AppConfig.java         # 全局 App 配置
│       ├── BxtConstant.kt         # 全局常量
│       ├── BxtenApp.kt            # Application 入口
│       ├── CommonComponent.java   # 通用 DI Component
│       ├── CrashHandler.java      # 崩溃捕获
│       ├── GlobalAppComponent.java # 全局 DI Component
│       │
│       ├── app/                   # 主 App 业务层（登录/首页/设置等）
│       ├── business/              # 核心业务模块
│       │   ├── activity/          # Activity 页面
│       │   ├── adapter/           # RecyclerView Adapter
│       │   ├── bean/              # 数据实体类
│       │   ├── broker/            # 券商/经纪人模块
│       │   ├── contract/          # 合约/期货交易模块
│       │   │   ├── ContractFragment.kt       # 合约主页 Tab
│       │   │   ├── ContractMarketDetailActivity.kt  # 合约行情详情
│       │   │   ├── DTContractFragment.kt     # DT 合约 Fragment
│       │   │   ├── NContractCurrentEntrustActivity.kt  # 当前委托
│       │   │   ├── NContractHistoryEntrustActivity.kt  # 历史委托
│       │   │   ├── NContractTradeFragment.kt  # 合约交易 Fragment
│       │   │   └── NPositionFragment.kt       # 持仓 Fragment
│       │   ├── dialog/             # 对话框
│       │   ├── fragment/           # Fragment 页面
│       │   ├── home/               # 首页模块
│       │   ├── kline/              # K线图模块
│       │   ├── otc/                # OTC 场外交易模块
│       │   ├── redpackage/         # 红包模块
│       │   ├── treaty/             # 协议/法律文书
│       │   └── view/               # 自定义 View
│       │
│       ├── corporealNet/           # 有价资产（充提币）模块
│       ├── db/                    # 数据库模块（Room/SQLite）
│       ├── entity/                # 实体类
│       ├── extra/                 # 扩展功能
│       ├── futures/               # 期货模块（部分合约相关）
│       ├── interceptor/           # OkHttp 拦截器
│       ├── manager/               # 管理器（SocketManager 等）
│       ├── model/                 # 数据 Model
│       ├── net/                   # 网络层
│       │   ├── DataHandler.kt      # 数据处理器
│       │   ├── HttpClient.kt       # HTTP 客户端封装
│       │   └── api/                # API 接口定义
│       └── util/                   # 工具类
│
├── followordersdk/               # 跟单 SDK（独立模块）
│   └── src/main/java/com/follow/order/
│       ├── contract/              # 跟单合约 Presenter
│       ├── model/
│       └── view/
│
├── dsbridge/                    # DsBridge（JSBridge 桥接库）
├── httplib/                     # HTTP 封装库
├── idcard/                      # 身份证识别模块
├── logcollector/                # 日志收集 SDK
├── motion/                      # 活动/运营模块
├── pushlib/                     # 推送 SDK
├── repositories/                # Maven 私有仓库
├── wslib/                       # WebSocket 封装库
├── dimenlibrary/                # 屏幕适配 dimension 库
│
├── build.gradle                 # 根构建脚本
├── config.gradle                # 全局配置（SDK版本/依赖版本）
├── appchange.gradle             # 多渠道/多环境配置
└── gradle.properties            # Gradle 配置
```

#### 核心 API 模块

```
net/api/
├── ApiConstants.java            # API 地址常量（含多环境：test/prod）
├── ApiService.kt                # 现货/OTC 相关 Retrofit API
├── HttpResult.java              # HTTP 统一响应封装
└── WebApiService.kt             # Web API 服务
```

#### 关键配置说明

- **多渠道配置**（`appchange.gradle`）：通过 `change.applicationId` 支持不同包名（如 `com.quick.bbex`）
- **多环境 URL**：`basebbrUrl`（现货）、`otcbbrBaseUrl`（OTC）、`contractbbrUrl`（合约）、`brokerbbrUrl`（券商）等分别配置
- **WebSocket 地址**：`socketbbrAddress`（现货）、`otcbbrSocketAddress`（OTC聊天）、`contractbbrSocketAddress`（合约推送）

#### 功能定位

exchangeAndroid 是面向 C 端用户的**综合交易 App**，集成了：
- 现货交易（Exchange）
- 合约/期货交易（Contract/Futures）
- OTC 场外交易
- 充提币管理
- 券商/经纪人功能
- 红包功能

---

### 2.2 exchangeContract（合约管理后台 - Vue）

#### 技术栈

| 类别 | 技术 |
|---|---|
| 框架 | Vue 2.5.x |
| UI 组件库 | CoreUI Pro（Bootstrap 4） |
| 图表 | Chart.js + vue-chartjs + ECharts 5.1 |
| 构建工具 | @vue/cli-service 3.3 |
| 状态管理 | Vuex 3.x |
| 路由 | Vue Router 3.x |
| HTTP | Axios |
| 国际化 | vue-i18n 8.x |
| 地图 | vue2-google-maps |
| 表单验证 | Vuelidate |
| 编辑器 | Quill（富文本）/ CodeMirror |
| 布局 | vue-grid-layout |

#### 目录结构

```
exchangeContract/
├── src/
│   ├── main.js                   # Vue 入口
│   ├── App.vue                   # 根组件
│   ├── apis/                     # API 接口定义
│   ├── assets/                   # 静态资源
│   ├── components/               # 公共组件
│   │   ├── chaets/              # 图表相关组件
│   │   ├── date.vue             # 日期选择组件
│   │   ├── dateSingle.vue       # 单日期选择
│   │   ├── eyePassword.vue      # 密码显示切换
│   │   ├── getCode.vue          # 获取验证码按钮
│   │   ├── pagination.vue       # 分页组件
│   │   └── realStatistics/      # 实时统计组件
│   │
│   ├── locale/                  # 国际化语言文件
│   ├── mixin/                   # Vue Mixin（混入）
│   ├── polyfill.js              # 浏览器兼容性垫片
│   ├── router/                  # 路由配置
│   │
│   ├── viewsTemplate/           # 模板页面（可复用）
│   │   ├── common/             # 通用页面
│   │   ├── futuresExchange/    # 期货交易所管理
│   │   ├── marketManagement/   # 市场管理
│   │   ├── merchantFollow/     # 商户跟随
│   │   ├── monitoring/         # 监控面板
│   │   ├── reportManager/      # 报表管理
│   │   ├── systemManagement/    # 系统管理
│   │   └── userData/           # 用户数据
│   │
│   ├── viewsCustom/             # 定制化业务页面
│   │   ├── components/         # 定制组件
│   │   ├── error_500.vue      # 500 错误页
│   │   └── login.vue           # 登录页
│   │
│   └── vuex/                   # Vuex 状态管理
│
├── public/                      # 静态公共资源
├── package.json
├── vue.config.js
├── babel.config.js
├── jest.config.js               # 单元测试配置
├── nginx.conf                   # Nginx 部署配置
├── Dockerfile
└── README.md
```

#### 功能定位

exchangeContract 是面向 **运营人员/管理员** 的合约/期货交易**管理后台**，主要功能包括：
- 期货交易所管理（futuresExchange）
- 市场管理（marketManagement）
- 商户/经纪人管理（merchantFollow）
- 系统管理与用户数据（systemManagement/userData）
- 实时监控与报表（monitoring/reportManager）

> 这是基于 CoreUI Pro Vue 模板构建的**可二次开发**的后台管理系统框架。

---

### 2.3 exchangeUi-bank（银行 UI 组件库/模板 - Vue）

#### 技术栈

| 类别 | 技术 |
|---|---|
| 框架 | Vue 2.5.x |
| UI 组件库 | CoreUI Pro（Bootstrap 4） |
| 构建工具 | @vue/cli-service 3.3 |
| 状态管理 | Vuex 3.x |
| 路由 | Vue Router 3.x |
| HTTP | Axios |
| 国际化 | vue-i18n 8.x |

#### 目录结构

```
exchangeUi-bank/
├── H5/                         # 移动端 H5 页面
│   ├── pages/
│   │   ├── app-mixin.js        # App Mixin
│   │   ├── co-home/            # 合约首页
│   │   ├── ex-home/            # 交易所首页
│   │   ├── otc-home/           # OTC 首页
│   │   ├── pages.styl          # 页面样式
│   │   └── pages.js            # 页面配置
│   ├── components/             # H5 组件
│   ├── common-mixin/           # 通用 Mixin
│   │   └── contract/          # 合约相关 Mixin
│   ├── router/                 # 路由
│   └── utils/                  # 工具
│
├── PC/                         # PC Web 页面
│   ├── pages/
│   │   ├── app-mixin.js
│   │   ├── co-home/            # 合约（PC端）
│   │   ├── ex-home/            # 交易所（PC端）
│   │   ├── otc-home/           # OTC（PC端）
│   │   ├── pages.js
│   │   └── pages.styl
│   ├── components/
│   ├── common-mixin/
│   │   └── contract/
│   ├── router/
│   └── vuex/
│
├── lib/                        # 公共库
├── node/                       # Node 工具脚本
├── static/                     # 静态资源
├── web-worker/                 # Web Worker
├── webpack-plugin/             # 自定义 Webpack 插件
├── websocket/                  # WebSocket 封装
│
├── emails.js                   # 邮件模板
├── home/                       # 首页相关
│   ├── getLocales.js           # 国际化获取
│   ├── publictInfo.js          # 公共信息
│   └── skin.js                 # 皮肤配置
├── router/                     # 路由
├── utils/                      # 工具函数
├── npmPublish.js               # NPM 发布脚本
│
├── package.json
├── babel.config.js
├── postcss.config.js
├── vue.config.js
└── yarn.lock
```

#### 功能定位

exchangeUi-bank 是一套面向 **银行/金融类业务** 的 **H5 + PC 双端 UI 组件库和页面模板**，提供：
- PC 端管理页面模板
- H5 端移动页面模板
- 合约/OTC/现货交易等多业务线 UI 复用组件
- 皮肤/主题切换能力（skin.js）
- 国际化支持

它与 exchangeContract 结构类似，但定位不同——bank 版本偏向金融/银行类 UI，contract 版本偏向合约交易管理。

---

### 2.4 exchangeweb-bank（银行 Web 平台后端 - Egg.js）

#### 技术栈

| 类别 | 技术 |
|---|---|
| 后端框架 | Egg.js 2.x（基于 Koa） |
| 前端渲染 | Vue 2.5 + Nunjucks 模板 |
| 前端 UI | Ant Design Vue 1.7 |
| HTTP 客户端 | Axios |
| 图表 | ECharts |
| 国际化 | Vue I18n |
| 文件处理 | xlsx、html2canvas、file-saver |
| KYC | @sumsub/websdk（身份验证） |
| 测试 | Mocha |
| 部署 | Docker |

#### 目录结构

```
exchangeweb-bank/
├── app/
│   ├── controller/              # 控制器（路由处理）
│   │   ├── fePublicInfo.js      # 公共信息接口
│   │   ├── home.js             # 首页接口
│   │   └── serverUrl.js        # 服务端 URL 配置
│   │
│   ├── service/                 # 业务逻辑层
│   │   ├── getLocales.js       # 国际化数据服务
│   │   ├── publictInfo.js      # 公共信息服务
│   │   └── skin.js             # 皮肤服务
│   │
│   ├── router.js               # 路由配置
│   ├── view/                   # Nunjucks 模板视图
│   ├── public/                 # 静态资源
│   ├── middleware/             # 中间件
│   ├── schedule/              # 定时任务
│   └── utils/                 # 工具函数
│
├── config/
│   ├── config.default.js      # 默认配置
│   ├── config.prod.js         # 生产环境配置
│   └── plugin.js              # 插件配置
│
├── test/                       # 测试用例
├── logs/                       # 日志目录
├── run/                        # 运行时 PID 文件
├── typings/                    # TypeScript 类型定义
├── serverConfig.json           # 服务端配置文件
│
├── appveyor.yml               # CI 配置
├── package.json
├── babel.config.js
├── jsconfig.json
├── vue.config.js
└── Dockerfile-Ex / Dockerfile-Otc   # 多环境 Docker 配置
```

#### 功能定位

exchangeweb-bank 是面向 **银行/OTC 业务** 的 **Web 平台后端服务**：
- 提供 REST API 给前端页面调用
- 使用 Nunjucks 模板引擎进行服务端渲染
- 支持 OTC（场外交易）业务
- 集成 KYC 身份验证服务（Sumsub）
- 定时任务处理（日报表生成等）
- 提供皮肤/多语言/主题等配置能力

---

### 2.5 futures（期货交易微服务 - Java）

#### 技术栈

| 类别 | 技术 |
|---|---|
| 语言 | Java 1.8 |
| 框架 | Spring Boot 2.1.5.RELEASE |
| 构建工具 | Maven（多模块） |
| 数据库 | MySQL + Sharding-JDBC 3.1（分库分表） |
| ORM | MyBatis Plus 3.4 |
| 消息队列 | RocketMQ 4.6.1 / 4.7.1 |
| RPC | gRPC（Protobuf 3.9）+ Retrofit 2.9 |
| 网关 | Spring Cloud Gateway（Nacos 2.1.2） |
| 限流 | Sentinel |
| 分布式 ID | 百度 UidGenerator（百度 FSG） |
| JSON | FastJSON 1.2.74 |
| 日志 | Log4j2（2.23.1，修补漏洞版本） |
| 文档 | Swagger 2.9 |
| Excel | EasyPoi 4.1 + xlsx-streamer |

#### 模块架构

```
futures/（Maven 多模块项目）
│
├── futures-parent/             # 父 Module，统一依赖版本管理
│   └── pom.xml
│
├── futures-common/             # 公共模块
│   └── pom.xml                # 被所有服务依赖
│
├── futures-base/              # 基础数据服务（用户/配置/券商）
│   ├── futures-base-client/    # 客户端（接口定义/Entity）
│   │   └── src/main/java/com/chainup/futures/base/entity/
│   │       ├── ConfigCoinSymbol.java       # 币种配置
│   │       ├── ConfigSymbolMatching.java  # 交易对配置
│   │       ├── ConfigSymbolMatchingMapper
│   │       ├── ContractConfigLadderMapper # 阶梯保证金配置
│   │       ├── ContractConfigLeverMapper  # 杠杆配置
│   │       ├── SaasUserVo.java            # SaaS 用户视图
│   │       └── params/                    # 请求参数类
│   │
│   └── futures-base-service/   # 服务端实现
│       └── src/main/java/com/chainup/futures/base/
│           ├── FuturesBaseServiceApplication.java
│           ├── repository/                    # MyBatis Mapper
│           │   ├── ConfigCoinMapper.java
│           │   ├── ConfigSymbolMapper.java
│           │   ├── ContractConfigMapper.java
│           │   ├── FuturesBrokerMapper.java   # 券商经纪人
│           │   ├── UserMapper.java
│           │   └── AdminUserMapper.java
│           └── service/
│
├── futures-api/                # API 定义模块（接口 + DTO）
│   └── pom.xml
│
├── futures-id/                 # 分布式 ID 生成服务
│   └── id-generator/
│       └── src/main/java/com/chainup/futures/utils/id/
│           ├── UidGenerator.java           # ID 生成器接口
│           ├── BitsAllocator.java           # 位分配器（Snowflake 变体）
│           ├── impl/
│           │   ├── DefaultUidGenerator.java
│           │   └── CachedUidGenerator.java  # 缓存型（性能优化）
│           ├── worker/
│           │   ├── WorkerNodeDAO.java       # Worker 节点管理
│           │   └── DisposableWorkerIdAssigner.java
│           └── buffer/
│               └── RejectedPutBufferHandler.java
│
├── futures-index/              # 指数/标记价格服务
│   ├── futures-index-client/
│   │   └── src/main/java/com/chainup/futures/index/
│   │       ├── client/
│   │       │   ├── IndexRpcClient.java      # 指数价格 RPC 客户端
│   │       │   └── FundRateRpcClient.java   # 资金费率 RPC 客户端
│   │       ├── dto/
│   │       │   └── TagIndexPrice.java        # 标记/指数价格 DTO
│   │       └── entity/params/
│   │           ├── IndexPriceKlineParams.java
│   │           └── UnderlyingPriceManageParams.java
│   │
│   └── futures-index-service/
│       └── src/main/java/com/chainup/futures/index/
│           ├── dto/                        # 数据传输对象
│           │   ├── BinanceTickerParam.java  # Binance 行情
│           │   ├── HuobiTickerParam.java    # 火币行情
│           │   ├── OkexTickerParam.java      # OKEx 行情
│           │   └── CoinbaseTickerParam.java  # Coinbase 行情
│           └── repository/
│               ├── IndexPriceMapper.java       # 指数价格
│               ├── IndexPriceKlineMapper.java # 指数 K 线
│               ├── TagPriceKlineMapper.java    # 标记价格 K 线
│               └── UnderlyingPriceManageMapper.java
│
├── futures-gateway/            # API 网关服务
│   ├── futures-gateway-service/
│   │   └── src/main/java/com/chainup/futures/gateway/
│   │       ├── ChainupGatewayApplication.java
│   │       ├── filter/                    # Gateway Filter
│   │       │   ├── AppSignGatewayFilterFactory.java     # App 签名验证
│   │       │   ├── CompanyCheckGatewayFilterFactory.java # 公司验证
│   │       │   ├── LogRequestGlobalFilter.java          # 请求日志
│   │       │   ├── IpGatewayFilterFactory.java          # IP 过滤
│   │       │   └── limit/                     # 限流
│   │       │       ├── IPLimitRateGlobalGateway.java
│   │       │       ├── UIDLimitRateGlobalGateway.java
│   │       │       ├── UserPathLimitRateGlobalFilter.java
│   │       │       └── IPPathLimitRateGlobalFilter.java
│   │       ├── config/
│   │       │   └── FastJsonMessage.java       # FastJSON 消息转换
│   │       ├── handler/
│   │       │   └── ExceptionHandler.java       # 全局异常处理
│   │       ├── datasource/
│   │       │   └── SentinelDataSource.java    # Sentinel 数据源
│   │       ├── entity/
│   │       │   └── ResultGateway.java         # 网关统一响应
│   │       ├── constants/
│   │       │   ├── APIConstants.java          # API 常量
│   │       │   └── LimitRateConstants.java    # 限流常量
│   │       └── utils/
│   │           ├── ResultUtils.java           # 响应工具
│   │           ├── IPUtil.java                 # IP 工具
│   │           └── MD5Utils.java              # MD5 工具
│   │
│   ├── futures-token-server/   # Token 服务
│   │   └── pom.xml
│   └── pom.xml
│
├── futures-order/              # 订单/交易服务
│   ├── futures-order-client/
│   │   └── src/main/java/com/chainup/futures/order/entity/
│   │       ├── CoOrder.java              # 订单实体
│   │       ├── CoTrade.java              # 成交记录
│   │       ├── CoPosition.java           # 持仓实体
│   │       ├── ExTransfer.java           # 划转记录
│   │       ├── TriggerOrder.java         # 触发单（止盈止损）
│   │       ├── LimitTriggerOrder.java   # 限价触发单
│   │       ├── CloudTransfer.java       # 云端划转
│   │       ├── SendCoinRecord.java      # 提币记录
│   │       ├── BrokerFeeShare.java       # 券商手续费分成
│   │       └── rpcParam/                 # RPC 参数类
│   │           ├── OrderParam.java
│   │           ├── TradeParam.java
│   │           ├── PositionParam.java
│   │           └── TriggerParam.java
│   │
│   ├── futures-order-service/
│   │   └── src/main/java/com/chainup/futures/order/
│   │       ├── service/impl/            # 业务实现
│   │       ├── repository/               # Mapper
│   │       ├── rpc/                      # RPC 调用
│   │       └── config/                   # 配置
│   └── pom.xml
│
├── futures-liquidation/        # 强平/清算服务
│   ├── futures-liquidation-client/
│   │   └── src/main/java/com/chainup/futures/liquidation/entity/
│   │       ├── LiquidationVo.java            # 清算视图
│   │       ├── LiquidationRecordVo.java       # 清算记录
│   │       ├── LiquidationPositionRecord.java # 持仓清算记录
│   │       ├── UPositionChange.java           # 持仓变化
│   │       └── PositionScanRecord.java        # 持仓扫描记录
│   │
│   └── futures-liquidation-service/
│       └── src/main/java/com/chainup/futures/liquidation/
│           └── service/impl/
│
├── futures-settlement/          # 结算服务（资金费用/定期结算）
│   ├── futures-settlement-client/
│   │   └── src/main/java/com/chainup/futures/settlement/entity/
│   │       ├── ContractConfig.java         # 合约配置
│   │       ├── SettlementVo.java           # 结算视图
│   │       ├── CoPosition.java            # 持仓（结算用）
│   │       ├── SettlementRiskRecord.java   # 风险结算记录
│   │       └── HistoryFundingRate.java     # 历史资金费率
│   │
│   └── futures-settlement-service/
│       └── src/main/java/com/chainup/futures/settlement/
│           ├── FuturesSettlementServiceApplication.java
│           ├── job/
│           │   └── SettlementJob.java       # 结算定时任务
│           ├── service/impl/
│           │   ├── SettlementServiceImpl.java       # 结算服务
│           │   ├── FundingRateServiceImpl.java     # 资金费率服务
│           │   └── PublicServiceImpl.java           # 公共接口
│           └── repository/
│               ├── CoPositionMapper.java
│               ├── HistoryFundingRateMapper.java
│               └── SettlementRiskRecordMapper.java
│
├── futures-job/                 # 定时任务服务
│   └── src/main/java/com/chainup/futures/job/
│       └── (定时任务实现)
│
└── futures-open-ws/             # WebSocket 服务（行情/交易推送）
    └── (WebSocket 处理)
```

---

## 三、项目间依赖关系与调用关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        C 端用户                                  │
│                                                                   │
│   ┌─────────────────────┐    HTTP/WebSocket                     │
│   │  exchangeAndroid     │ ◄──────────────────────────────────│
│   │  (Kotlin Android)   │                                      │
│   └──────────┬───────────┘                                      │
│              │ REST API + WebSocket                               │
└──────────────┼───────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────────┐
│                     外部/内部 API 网关层                          │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │              futures-gateway（Spring Cloud Gateway）        │  │
│  │  • 请求签名验证（AppSignGatewayFilter）                       │  │
│  │  • 公司/用户身份验证（CompanyCheckGatewayFilter）              │  │
│  │  • IP/UID/路径限流（Sentinel + 自实现）                       │  │
│  │  • 请求日志（LogRequestGlobalFilter）                        │  │
│  └──────────────────────────┬───────────────────────────────────┘  │
└─────────────────────────────┼────────────────────────────────────┘
                              │ RPC（gRPC/Retrofit）
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│ futures-base │    │ futures-order│    │ futures-index     │
│ 基础数据服务  │    │ 订单交易服务  │    │ 指数/标记价格服务  │
│              │    │              │    │                  │
│ • 用户管理    │    │ • 下单/改单   │    │ • 从 Binance/     │
│ • 合约配置    │    │ • 成交记录    │    │   Huobi/OKEx     │
│ • 币种配置    │    │ • 持仓管理    │    │   获取行情        │
│ • 券商经纪人  │    │ • 触发单      │    │ • 计算指数价格    │
│ • 短信       │    │ • 划转       │    │ • 资金费率        │
└──────┬───────┘    └──────┬───────┘    └────────┬─────────┘
       │                   │                     │
       └───────────────────┼─────────────────────┘
                           │ 消息队列（RocketMQ）
                           ▼
              ┌────────────────────────┐
              │  futures-liquidation    │
              │  强平/清算服务           │
              │  futures-settlement     │
              │  结算服务               │
              │  futures-job            │
              │  定时任务              │
              └────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     管理端（运营/Admin）                          │
│                                                                   │
│   ┌──────────────────────┐  HTTP/REST                            │
│   │  exchangeContract    │ ◄──── 运营人员（期货/合约管理后台）     │
│   │  (Vue + CoreUI Pro)  │                                       │
│   └──────────────────────┘                                       │
│                                                                   │
│   ┌──────────────────────┐  HTTP/REST                            │
│   │  exchangeUi-bank     │ ◄──── 运营人员（银行类 UI）            │
│   │  (Vue + CoreUI Pro)  │                                       │
│   └──────────────────────┘                                       │
└───────────────────────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────────┐
│              exchangeweb-bank（Egg.js 后端）                       │
│  • 提供管理端 API（OTC/银行业务）                                   │
│  • Nunjucks 模板渲染                                               │
│  • KYC 身份验证集成（Sumsub）                                       │
│  • 定时任务（日报表等）                                             │
└───────────────────────────────────────────────────────────────────┘
```

### 详细依赖说明

| 调用方向 | 协议 | 说明 |
|---|---|---|
| Android App → futures-gateway | HTTP REST + WebSocket | 交易请求、行情订阅 |
| exchangeContract → futures-gateway | HTTP REST | 管理后台数据操作 |
| exchangeweb-bank → futures-gateway | HTTP REST | 银行/OTC 业务 API |
| futures-gateway → 各微服务 | gRPC / HTTP（Retrofit） | 内部服务调用 |
| futures-order → futures-index | RPC | 获取标记价格/指数价格 |
| futures-order → futures-base | RPC | 获取用户信息/合约配置 |
| futures-liquidation → futures-order | RocketMQ 消息 | 持仓变化事件驱动 |
| futures-settlement → futures-order | RocketMQ 消息 | 结算触发 |
| futures-job → 各服务 | RPC | 定时任务触发 |

---

## 四、整体架构总结

### 4.1 架构风格

这是一个典型的 **前后端分离 + 微服务** 架构：

- **前端**：Android（Kotlin）、Vue.js Web（H5 + PC）
- **后端**：Java 微服务（Spring Boot）+ Node.js/Egg.js
- **通信**：REST HTTP + WebSocket + gRPC + RocketMQ

### 4.2 业务线划分

```
交易所业务
├── 现货交易（Exchange）──── exchangeAndroid（app 模块）+ exchangeweb-bank（OTC）
├── 合约交易（Contract）──── exchangeAndroid（contract 模块）+ futures（微服务）
└── 银行/OTC（Bank）────── exchangeUi-bank + exchangeweb-bank
```

### 4.3 技术特点

**优点/特点：**
1. **多端覆盖**：Android + H5 + PC Web + 管理后台，全端产品线
2. **微服务拆分**： futures 项目将业务按领域合理拆分为 10+ 微服务
3. **多维度限流**：网关层实现了 IP/UID/路径三级限流
4. **分布式 ID**：采用百度 UidGenerator 解决 ID 生成问题
5. **分库分表**：使用 Sharding-JDBC 处理数据水平拆分
6. **多源行情**：futures-index 从多个交易所（Binance/Huobi/OKEx/Coinbase/Bitfinex）聚合行情

**潜在风险：**
1. **FastJSON 安全**：1.2.74 版本存在已知安全漏洞（已引入但需关注）
2. **Log4j2 版本**：2.23.1 为修漏洞版本，需持续关注
3. **Spring Boot 2.1.5**：较老的小版本，部分依赖可能已停止维护
4. **Vue 2 + CoreUI Pro**：CoreUI Pro 为商业闭源组件，定制需有源码
5. **MySQL + Sharding-JDBC**：分库分表后跨表查询复杂度增加

### 4.4 典型请求流程（合约下单）

```
1. 用户在 Android App 点击"买入"
2. exchangeAndroid/net/api/ApiService.kt 发起 HTTP 请求
3. 请求到达 futures-gateway（API 网关）
   - AppSignGatewayFilterFactory 验证签名
   - CompanyCheckGatewayFilterFactory 验证公司权限
   - IPLimitRateGlobalGateway 限流检查
4. Gateway 通过 gRPC 转发到 futures-order-service
5. futures-order-service：
   - 调用 futures-index 获取当前标记价格
   - 调用 futures-base 获取合约配置（保证金率等）
   - 创建订单（CoOrder）并记录
6. 通过 RocketMQ 发送消息给 futures-liquidation
7. futures-liquidation 扫描持仓，执行强平检查
8. WebSocket 推送订单状态变更给 Android 客户端
```

---

## 五、文件输出信息

- **分析报告路径**：`~/openclaw_projects/exchange/analysis.md`
- **分析时间**：2026-04-09 13:07 GMT+8
- **分析深度**：目录结构 + 关键模块/类/配置文件
- **备注**：项目中未发现 Solidity 智能合约或 Truffle/Hardhat 配置，"Contract" 在本项目中指合约/期货交易业务模块
