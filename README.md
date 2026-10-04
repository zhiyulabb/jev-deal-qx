# Jev 购物分析 · Quantumult X

1. **配置 Jev。** 在 [Typesafe 控制台](https://console.typesafe.ai) 获取 key，在现有 `[task_local]` 段加入 `0 0 * * * https://raw.githubusercontent.com/zhiyulabb/jev-deal-qx/main/jev_deal.js#setup-key=YOUR_API_KEY, tag=Jev API配置, enabled=true`，仅在手机替换 key。向右滑动任务执行一次，看到“API key 已保存”后删除任务。真实 key 只保存在本机；不要分享含 key 的配置。公开脚本保持占位符。
2. **引用规则、获取历史价配置。** 在圈 X 重写引用中添加 `https://raw.githubusercontent.com/zhiyulabb/jev-deal-qx/main/jev_deal.conf`，开启 MitM 并安装、信任证书；解密域名以 conf 为准。打开慢慢买 App「我的」，收到“慢慢买配置已保存”即可。配置只用于慢慢买请求，不传给 Jev。京东与淘宝均尝试商品映射、走势、价格摘要；失败回退本地浏览记录，不伪造历史价。第三方历史缓存6小时。匹配商品编号是为防止价格串商品，不会在批量响应里随意取第一条价格。
3. **进入商品并检查。** 京东商品接口可直接读取 `floors[].data.priceInfo.jprice`，批量 `mGetsByColor` 价格按商品缓存60秒；H5 商品首页及图文页也能触发，不要求先点击图文详情。淘宝加入 `mtop.taobao.detail.getdetail` 和 `mtop.taobao.detail.data.get`，解析 `apiStack[].value` 及 `global.data`。淘宝未选择规格时只接受商品级单一数值展示价，区间价格不猜测、不选随机 SKU。通知按商品名称、Jev 决策分析、历史价格排列，无空行；证据与优惠评分分别标注，未核实抬价时显示待核。历史低、180天低、60天低、30天低、618及双11均单独显示价格与日期；缺失项目不编造。保留模型动作，不因缺少本地浏览记录一律覆盖成“信息不足”。通知的证据0–5分由当前平台价、明确规格、可用历史、当前到手条件已核实、历史同规格同条件已核实五项分别计1分；优惠0–5分是模型对优惠强度的评分，两者独立。未核实条件的购买结果标为“参考购买”。只有至少三条本地观察且当前条件已核实时才显示抬价估计，否则显示“待核”，并不代表证实抬价。现有采集不自动宣称到手及历史条件已核实，因此不能保证证据满分。Jev 的三个结构化结果依据[官方 API](https://docs.typesafe.ai/api)，说明由脚本翻译，非模型生成自由文本。京东原始字段参考 [JDHelloWorld](https://github.com/JDHelloWorld/jd_price/blob/main/jd_price.js)，淘宝嵌套数据参考 [yichahucha](https://github.com/yichahucha/surge/blob/master/tb_price.js)，新接口参考 [Toperlock](https://github.com/Toperlock/Quantumult/blob/02d6b4eed022541d88256e3924f4ce531a0e6913/Rewrites/Scripts/jd_tb_price.js)。只采用解析方式，没有修改页面、HTTPDNS 或证书机制。模拟回放已检查京东数组、入口触发、淘宝新旧接口/JSONP/区间、商品错配、历史请求及签名、缓存、凭据隔离、失败与超时；移动网页版实测找到首页内嵌 `_itemInfo.priceFloor.ext.jdPrice`，与页面展示54.90及39.90一致，已接入；解析兼容字符串含括号与对象末尾逗号。移动网页访问即可触发。京东原生 App 立即触发与淘宝无通知仍未解决，需实际请求确认。电脑直访京东价格接口实测403，公开页面价格被遮蔽；不能承诺抓到登录账号到手价。若 App 不发出已支持且可解密的接口，这版仍无法取得该价。第三方接口可能失效。所有退出原样 `$done({})`；历史请求每次最多2.5秒，总预算9.5秒，模型5秒，总保护15秒。必要时关闭引用及其新增解密主机。

### 历史价格与模型结果独立

Jev 未配置、请求失败或返回格式无效时，通知仍显示当前展示价和已取得的历史价。慢慢买返回商品编号不一致时，不引用其他商品价格；若本机有同一商品已验证的历史缓存，则显示缓存并标明本次刷新失败。没有有效记录时保留历史价格栏目及失败原因，不虚构历史数据。过期缓存中的第三方当前价不作为新的当前价格使用。

### 淘宝历史独立触发

淘宝详情返回可识别的商品编号后，即使当前价格缺失、为区间价、apiStack 无法解析或多个候选价格冲突，也独立查询同一商品的慢慢买历史。无当前价时不调用 Jev，不把慢慢买参考价当作淘宝账号价。兼容直接 data 和 apiStack/global.data；多个商品编号冲突时停止处理。未修改 AMDC 调度、请求或商品响应。

### 京东详情编号独立触发

参照 yichahucha、zZPiglet 的商品链接提取方式，兼容 floors、others.property.shareUrl 和 commonBaseInfo，以及 data 内的详情容器。取得明确商品编号后即使缺少价格，也进入独立历史查询；多个编号冲突不引用，价格冲突只保留历史查询。配置预加载模板和推荐商品不作为当前商品。当前账号到手价仍需实际详情响应支持，慢慢买参考价单独标记。HTTPDNS 配置保持原样。

### 2026-10-03 浏览器脚本对照

核对 [购物党](https://greasyfork.org/zh-CN/scripts/436876/code) 与 [购物优惠券小助手](https://greasyfork.org/zh-CN/scripts/497783/code) 源码：前者在网页环境通过 `acs.m.taobao.com/h5/mtop.taobao.detail.getdetail/6.0/` 获取商品信息，其列表辅助逻辑取 `apiStack[0].value` 内 `skuCore.sku2info["0"].price.priceMoney / 100`，失败则退回网页采集价；这不证明是当前所选规格或账号券后价。后者通过 `api2.jasonzk.com/tools/goods-his` 按商品链接查询第三方历史价格，使用浏览器 DOM 展示。两者均不能直接作为圈 X 原生 App 抓取方案。现有规则已覆盖前者的详情接口，未额外接入默认 SKU 价格、推广转换接口或第三方当前价，避免把参考价标成账号价。此对照不代表淘宝 App 抓取问题已修复。

### 京东 PC 详情展示价补充

参考购物党网页数据结构，新增 `pc_detailpage_wareBusiness` 响应解析：从 GET 查询或 POST 表单提取请求 SKU，核对响应的 `pageConfigVO.skuid` / `skuId`，仅取同一响应的 `price.finalPrice.price` 或 `price.p`，标记为京东网页展示价。原价 `op` 不作为当前价；错误响应、编号冲突、区间值不接受。规则原有 api.m.jd.com 覆盖该入口，无需新增主机。没有主动重放签名请求，未伪造 h5st，也未修复无法捕获原生 App 响应的问题。通过新字段、GET/POST、编号错配、接口错误、区间价及历史/淘宝回归检查；尚未在手机实测。

### 京东商品入口响应触发

`wareBusiness` 兼容从 GET 查询 / POST 表单的 body.skuId 或 body.wareId 识别当前商品。详情响应缺少商品编号时，依据该单商品请求读取详情价并查询历史；无价格仍查询历史。请求编号冲突、响应编号冲突或错误状态不采用。不会读取推荐价格，也不会修改请求；请求阶段只保存商品身份并立即放行，不进行网络查询。必须捕获详情响应才能触发，未承诺无法解密的 App 连接可用。新增入口测试与既有京东、淘宝解析回归通过，手机实测待确认。

### 京东触发与取价修复（v30）

圈 X 监听网络请求，不监听 App 的页面点击。进入商品后只有发出已覆盖且可解密的商品请求，才会触发分析；收藏、评论、购物车或其他任意操作不应误触发商品分析。

- `api.m.jd.com` 的请求体阶段提取 `wareBusiness` / `pc_detailpage_wareBusiness` 的商品编号，按官方 [`sessionIndex`](https://github.com/crossutility/Quantumult-X/blob/master/sample.conf) 与响应关联，90秒有效，最多保留50条。只保存编号、接口名、URL散列和时间，不保存 Cookie、签名或请求原文；该阶段不等待历史或模型请求。
- 响应阶段优先提取同商品 `priceInfo.jprice`、PC `price.finalPrice.price` / `price.p`。图文页与移动网页首页仍支持响应触发；请求阶段不再执行十五秒的历史查询。商品编号冲突仍拒绝引用。
- 商品响应缺少价格或无法解析但请求编号明确时，最多用2.5秒查询该商品 `item.m.jd.com/product/编号.html`，只接受同编号 `_itemInfo.priceFloor.ext.jdPrice` 的完整数字，标为“京东网页”。它不是 App 当前账号到手价；匿名网页可能返回 `6?` 或要求登录，此时拒绝取价。没有主动重放 App 签名，也不修改 HTTPDNS 或证书机制。
- 分析缓存与通知分别处理。十分钟内复用有效分析结果；再次进入会重新显示，八秒内完全相同的重复通知合并。无当前价时历史仍独立查询，失败显示原因，不伪造价格。
- 脚本日志以 `[Jev] v30` 标识，区分商品请求、会话关联、响应取价、网页遮蔽及短时去重，不输出请求头或账号信息。更新后若仍无通知，先核对这些日志及新请求的重写命中。仅替换引用，避免同时启用多个旧版本。

验证包括实际网页响应回放、POST 请求与缺少请求体的响应关联、并发商品隔离、过期会话拒绝、缓存再次通知、匿名遮蔽价拒绝、原响应不修改，以及既有京东、淘宝与历史回归。手机 App 实时入口尚需更新后的抓取确认，回放通过不等于手机入口已验证。

### 2026-10-04 购物党对照优化（v31）

核对主脚本及 `vendor-gwdv2.js`：购物党当前 SKU 从京东网页 React Fiber 读取，其他 SKU 查询依赖页面 `PSign` 和设备上下文；圈 X 无此网页运行环境，因此不移植 React 读取、不重放签名请求。已有 PC 解析继续优先 `price.finalPrice.price`、其次 `price.p`，原价 `op` 不作当前价格。

移动网页内嵌 JSON 兼容带括号和不带括号的赋值。价格被遮蔽或缺失时仍保留同商品名称、编号与规格，供独立历史查询使用；不以分享文案或历史接口当前价替代京东账号价。PC 响应同时校验 `pageConfigVO.skuid` 与 `skuId`，任一冲突均拒绝取价。没有引入购物党跟踪、推广或占位走势。

新解析检查、实际网页回放、京东 PC/App 入口、会话关联、历史与淘宝回归通过。手机 App 实时触发未验证；如果不发出可解密且受支持的请求，仍不能仅凭本次解析优化取得价格。

### 2026-10-04 实际 PC 响应验证（v32）

电脑圈 X 实际捕获 `pc_detailpage_wareBusiness`：商品 100018607635 的 `price.finalPrice.price` 为 11.75，`price.p` 为 15.00，现有解析回放正确取得11.75。价格对象的 `price.id` 也纳入商品校验。此结果不代表手机 App 接口已验证。

订阅固定旧提交的 conf 无法自动获取新版；日常更新请使用 main 配置链接。发布的 conf 仍固定脚本提交，保证每版规则和脚本一致。更新后需在圈 X 中刷新引用；磁盘配置变更不等于运行中的配置已经重载。
