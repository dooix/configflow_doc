# 规则仓库

规则仓库是集中管理所有规则集的地方，方便在多个规则配置中复用相同的规则集。

## 主要功能

### 1. 规则类型

支持三种规则集类型：

**domain（域名规则）**
- 包含域名列表的规则集
- 支持精确域名和域名后缀匹配
- 最常用的规则类型

**ipcidr（IP段规则）**
- 包含 IP 地址段（CIDR）的规则集
- 用于基于 IP 的分流
- 支持 IPv4 和 IPv6

**classical（传统规则）**
- 包含多种规则类型混合的规则集
- 兼容 Clash 传统规则格式
- 可以包含 DOMAIN、DOMAIN-SUFFIX、IP-CIDR 等多种规则

### 2. 规则来源类型

**URL 类型**
- 从远程 URL 获取规则内容
- 支持标准的 Clash rule-provider 格式
- 适合使用开源规则集

**内容类型**
- 直接在界面中编辑规则内容
- 规则内容存储在本地数据库
- 适合自定义规则集

### 3. 添加规则

**添加 URL 类型规则：**
```
名称：微信直连
类型：domain
来源：URL
URL：https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/wechat.txt
```

**添加内容类型规则：**
```
名称：公司内网
类型：domain
来源：content
内容：
  - company.com
  - internal.company.com
  - mail.company.com
```

### 4. 批量导入规则

支持从 YAML 格式的 rule-providers 配置批量导入。

**YAML 格式示例：**
```yaml
rule-providers:
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    interval: 86400

  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    interval: 86400

  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    interval: 86400
```

导入后会自动创建三个规则：`reject`、`proxy`、`direct`

### 5. 规则引用跟踪

每个规则会显示被哪些规则配置引用：
- 显示引用数量
- 点击可查看引用详情
- 删除规则前会提示影响范围

### 6. 规则管理操作

**编辑规则**
- 修改规则名称
- 修改规则 URL 或内容
- 修改规则类型

**删除规则**
- 删除规则会同时删除规则文件
- 如果规则被引用，会警告

**启用/禁用规则**
- 禁用规则不会出现在生成的配置中
- 可以暂时禁用而不删除

**拖拽排序**
- 调整规则在仓库中的显示顺序
- 不影响规则配置中的优先级

## 使用场景

**场景1：使用开源规则集**
```
1. 添加 Loyalsoldier 规则集合
   - 直连规则：https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt
   - 代理规则：https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt
   - 拒绝规则：https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt

2. 在规则配置中引用
   - 创建规则集，从仓库选择
   - 配置对应的策略
```

**场景2：自定义企业规则**
```
名称：公司内网域名
类型：domain
来源：content
内容：
  - gitlab.company.local
  - jenkins.company.local
  - nexus.company.local
  - wiki.company.local
```

**场景3：批量导入规则**
```yaml
# 准备好 rule-providers YAML 配置
rule-providers:
  google:
    behavior: domain
    url: "https://example.com/google.txt"

  telegram:
    behavior: domain
    url: "https://example.com/telegram.txt"

  streaming:
    behavior: domain
    url: "https://example.com/streaming.txt"

# 点击批量导入，粘贴上述YAML
# 系统会自动创建三个规则
```
