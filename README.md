# Jev 购物分析 · Quantumult X

1. **配置 Jev。** 在 [Typesafe 控制台](https://console.typesafe.ai) 获取 key，在现有 `[task_local]` 段加入 `0 0 * * * https://raw.githubusercontent.com/zhiyulabb/jev-deal-qx/main/jev_deal.js#setup-key=YOUR_API_KEY, tag=Jev API配置, enabled=true`，仅在手机替换 key。向右滑动任务执行一次，看到“API key 已保存”后删除任务。真实 key 只保存在本机；不要分享含 key 的配置。公开脚本保持占位符。
2. **引用规则、获取历史价配置。** 在圈 X 重写引用中添加 `https://raw.githubusercontent.com/zhiyulabb/jev-deal-qx/main/jev_deal.conf`，开启 MitM 并安装、信任证书；解密域名以 conf 为准。打开慢慢买 App「我的」，收到“慢慢买配置已保存”即可。配置只用于慢慢买请求，不传给 Jev。京东与淘宝均尝试商品映射、走势、价格摘要；失败回退本地浏览记录，不伪造历史价。第三方历史缓存6小时。匹配商品编号是为防止价格串商品，不会在批量响应里随意取第一条价格。
3. **进入商品并检查。** 京东商品接口可直接读取 `floors[].data.priceInfo.jprice`，批量 `mGetsByColor` 价格按商品缓存60秒；H5 商品首页及图文页也能触发，不要求先点击图文详情。淘宝加入 `mtop.taobao.detail.getdetail` 和 `mtop.taobao.detail.data.get`，解析 `apiStack[].value` 及 `global.data`。淘宝未选择规格时只接受商品级单一数值展示价，区间价格不猜测、不选随机 SKU。通知没有空行和分隔标题：先结论，再展示价、历史低/30天低/活动价，最后以两行呈现 Jev 原始动作、证据评分、把握度与抬价估计。历史低、180天低、60天低、30天低、618及双11均单独显示价格与日期；缺失项目不编造。优惠条件或可比记录不足时，最终为“信息不足”；模型原始动作和最终结果分别标明。评分衡量优惠证据，不是商品质量；抬价估计没有完整走势支持时标注未核实。Jev 的三个结构化结果依据[官方 API](https://docs.typesafe.ai/api)，说明由脚本翻译，非模型生成自由文本。京东原始字段参考 [JDHelloWorld](https://github.com/JDHelloWorld/jd_price/blob/main/jd_price.js)，淘宝嵌套数据参考 [yichahucha](https://github.com/yichahucha/surge/blob/master/tb_price.js)，新接口参考 [Toperlock](https://github.com/Toperlock/Quantumult/blob/02d6b4eed022541d88256e3924f4ce531a0e6913/Rewrites/Scripts/jd_tb_price.js)。只采用解析方式，没有修改页面、HTTPDNS 或证书机制。模拟回放已检查京东数组、入口触发、淘宝新旧接口/JSONP/区间、商品错配、历史请求及签名、缓存、凭据隔离、失败与超时；移动网页版实测找到首页内嵌 `_itemInfo.priceFloor.ext.jdPrice`，与页面展示54.90及39.90一致，已接入；解析兼容字符串含括号与对象末尾逗号。移动网页访问即可触发。京东原生 App 立即触发与淘宝无通知仍未解决，需实际请求确认。电脑直访京东价格接口实测403，公开页面价格被遮蔽；不能承诺抓到登录账号到手价。若 App 不发出已支持且可解密的接口，这版仍无法取得该价。第三方接口可能失效。所有退出原样 `$done({})`；历史请求每次最多2.5秒，总预算9.5秒，模型5秒，总保护15秒。必要时关闭引用及其新增解密主机。

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
