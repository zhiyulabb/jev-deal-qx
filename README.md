# Jev 双11购物通知（待真机验证）

1. 在 https://console.typesafe.ai 获取 key。公开仓库保持 `jev_deal.js` 第一行 `apikey_xxx`；仓库地址为 https://github.com/zhiyulabb/jev-deal-qx 。真实 key 在设备上通过一次性本地脚本写入：`$prefs.setValueForKey("你的真实key", "jev:api_key"); $done({});`，执行后删除该本地配置脚本，不上传、不打印 key。当前仅使用近90天本地浏览价格记录；没有接通第三方历史价，不保证完整历史或最低市场价。通知按 Jev 结果、当前价格、历史价格与来源排列。商品文案之外只发送提取字段及价格记录，不转发 Cookie、账号或原始响应。商品信息会发送给 TypeSafe。
2. 圈 X「重写 → 引用」填写 `https://raw.githubusercontent.com/zhiyulabb/jev-deal-qx/main/jev_deal.conf`，启用重写并更新资源。规则严格保留给定格式，实际 App 版本可能不命中；解析最多六层，只接受有标题、有效价格和商品 ID 的数据，淘宝还要求 SKU ID，无法对应时跳过。首次历史不足的 Jev 结果固定为“信息不足”；动作置信度低于 0.55 同样处理。成功商品标题与现价缓存命中且身份一致时不再请求或通知；失败允许下次重试。缓存与历史分别保存，重复浏览仍可积累记录。
3. MitM 启用 `api.m.jd.com`、`trade-acs.m.taobao.com`，安装并信任圈 X 证书，分流不用改。脚本最多等待5秒，所有路径 `$done({})`，不修改价格、库存、券或响应 body；脚本等待可能延迟响应。若商品页受 MitM 影响，关闭该引用及这两个主机的解密，不绕证书。上线验收仍需手机实际响应、自己的 API key，以及真实仓库 raw 链接：确认脚本能拉取、通知仅一条、响应不变、失败放行和重复请求去重。
