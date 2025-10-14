# 规则配置

规则配置模块用于创建具体的分流规则，支持单条规则和规则集引用。

## 主要功能

### 1. 单条规则类型

支持 14 种规则类型：

#### 域名类规则

**DOMAIN（精确匹配）**
- 精确匹配完整域名
- 示例：`www.google.com` 只匹配 `www.google.com`，不匹配 `mail.google.com`

**DOMAIN-SUFFIX（后缀匹配）**
- 匹配域名及其所有子域名
- 最常用的规则类型
- 示例：`google.com` 匹配 `google.com`、`www.google.com`、`mail.google.com`

**DOMAIN-KEYWORD（关键字匹配）**
- 匹配域名中包含关键字的域名
- 示例：`google` 匹配 `www.google.com`、`google.co.jp`、`googleusercontent.com`

#### IP类规则

**IP-CIDR（IPv4地址段）**
- 匹配 IPv4 地址段
- 使用 CIDR 表示法
- 示例：`192.168.0.0/16`、`10.0.0.0/8`

**IP-CIDR6（IPv6地址段）**
- 匹配 IPv6 地址段
- 示例：`2001:db8::/32`

**IP-SUFFIX（IP后缀）**
- 匹配 IP 地址后缀
- 示例：`8.8.8.8`

#### 端口类规则

**DST-PORT（目标端口）**
- 匹配目标端口
- 示例：`80`、`443`、`8080`

**SRC-PORT（源端口）**
- 匹配源端口
- 较少使用

#### 地理位置规则

**GEOIP（地理位置IP）**
- 根据 IP 地理位置匹配
- 示例：`CN`（中国）、`US`（美国）、`JP`（日本）

**GEOSITE（地理位置站点）**
- 根据域名地理属性匹配
- 需要 GeoSite 数据库支持
- 示例：`google`、`cn`、`geolocation-!cn`

#### 规则集引用

**RULE-SET（规则集）**
- 引用规则仓库中的规则集
- 示例：`proxy`、`direct`、`reject`

#### 逻辑规则

**AND（与逻辑）**
- 所有子规则都匹配时才匹配
- 示例：`AND,((DOMAIN,google.com),(DST-PORT,443))`

**OR（或逻辑）**
- 任一子规则匹配即匹配
- 示例：`OR,((DOMAIN,google.com),(DOMAIN,youtube.com))`

**NOT（非逻辑）**
- 子规则不匹配时才匹配
- 示例：`NOT,((GEOIP,CN))`

#### 兜底规则

**MATCH（匹配所有）**
- 匹配所有请求
- 必须放在规则列表最后
- 相当于 `FINAL` 规则

### 2. 添加单条规则

**操作步骤：**
1. 点击"添加规则"按钮
2. 选择规则类型
3. 输入规则值
4. 选择应用策略（DIRECT、REJECT 或策略组）
5. 设置启用状态
6. 保存

**示例：**
```
类型：DOMAIN-SUFFIX
值：google.com
策略：✈️ 代理选择
状态：启用
```

### 3. 批量导入域名

快速批量添加域名后缀规则。

**操作步骤：**
1. 准备域名列表（每行一个）
2. 点击"批量导入域名"按钮
3. 粘贴域名列表
4. 选择策略
5. 确认导入

**域名列表示例：**
```
google.com
youtube.com
gmail.com
googleusercontent.com
gstatic.com
```

系统会自动为每个域名创建 DOMAIN-SUFFIX 规则。

### 4. 添加规则集

**从规则仓库选择：**
1. 点击"添加规则集"按钮
2. 从下拉列表选择规则仓库中的规则
3. 规则名称、URL、类型会自动填充
4. 选择应用策略
5. 保存

**手动输入：**
1. 点击"添加规则集"按钮
2. 清空规则选择（如果有）
3. 手动输入规则名称、URL、类型
4. 选择应用策略
5. 保存

**特点：**
- 从规则仓库选择的规则会关联仓库中的规则
- 规则仓库中的规则变更会自动同步
- 已被使用的规则仓库项不会在选择列表中重复出现

### 5. 规则排序

**拖拽排序：**
- 规则按照从上到下的顺序匹配
- 拖动规则卡片调整顺序
- 优先级高的规则应该放在前面
- 规则和规则集可以混合排序

**规则优先级原则：**
1. 特殊规则在前（DOMAIN 精确匹配）
2. 常用规则在中（DOMAIN-SUFFIX）
3. 通用规则在后（GEOIP）
4. MATCH 规则必须在最后

### 6. 规则集分组显示

**自动分组：**
- 相同策略的连续规则集会自动分组
- 分组卡片显示规则集数量和策略
- 点击分组可以展开查看详情

**展开/收起：**
- 收起状态：显示为一个分组卡片
- 展开状态：显示所有规则集卡片
- 收起按钮在展开组的第一个卡片上

**好处：**
- 简化界面，避免规则集过多时界面混乱
- 相同策略的规则集便于管理
- 拖拽时可以整组移动

### 7. 规则管理操作

**编辑规则/规则集**
- 修改规则类型、值、策略
- 规则集可以修改名称、URL、类型、策略

**删除规则/规则集**
- 删除规则配置
- 规则集删除会同步更新 MosDNS 配置中的引用
- 不会影响规则仓库中的原始规则

**启用/禁用**
- 禁用的规则不会出现在生成的配置中
- 可以临时禁用规则测试效果

## 使用场景

**场景1：构建完整的分流规则**

```
规则顺序（从上到下）：

1. 内网直连
   - IP-CIDR: 192.168.0.0/16 → DIRECT
   - IP-CIDR: 10.0.0.0/8 → DIRECT
   - IP-CIDR: 172.16.0.0/12 → DIRECT

2. 广告拒绝
   - 规则集：reject（来自规则仓库）→ REJECT

3. 国外服务代理
   - 规则集：proxy（来自规则仓库）→ ✈️ 代理选择
   - 规则集：google（来自规则仓库）→ ✈️ 代理选择
   - 规则集：telegram（来自规则仓库）→ ✈️ 代理选择

4. 国内服务直连
   - 规则集：direct（来自规则仓库）→ DIRECT
   - 规则集：wechat（来自规则仓库）→ DIRECT

5. 国内IP直连
   - GEOIP: CN → DIRECT

6. 其他流量
   - MATCH → ✈️ 代理选择
```

**场景2：特定网站的精细化分流**

```
# 将 Google 服务按地区分流
1. DOMAIN-SUFFIX: google.com.hk → 🇭🇰 香港节点
2. DOMAIN-SUFFIX: google.co.jp → 🇯🇵 日本节点
3. DOMAIN-SUFFIX: google.com → ✈️ 代理选择

# 将流媒体服务分流到特定节点
1. DOMAIN-SUFFIX: netflix.com → 🎬 流媒体
2. DOMAIN-SUFFIX: disneyplus.com → 🎬 流媒体
3. DOMAIN-SUFFIX: hulu.com → 🎬 流媒体
```

**场景3：批量导入常用网站**

```
批量导入域名列表：
google.com
youtube.com
facebook.com
twitter.com
instagram.com
github.com

选择策略：✈️ 代理选择

结果：自动创建6条 DOMAIN-SUFFIX 规则
```

**场景4：使用逻辑规则**

```
# HTTPS 的 Google 服务走代理
AND,((DOMAIN-SUFFIX,google.com),(DST-PORT,443)) → ✈️ 代理选择

# YouTube 或 Google 走代理
OR,((DOMAIN-SUFFIX,youtube.com),(DOMAIN-SUFFIX,google.com)) → ✈️ 代理选择

# 非中国 IP 走代理
NOT,((GEOIP,CN)) → ✈️ 代理选择
```

## 最佳实践

### 规则顺序
```
优先级从高到低：
1. 内网 IP 直连（192.168.0.0/16, 10.0.0.0/8）
2. 广告拦截规则集（REJECT）
3. 特殊服务规则（自定义）
4. 国外服务规则集（PROXY）
5. 国内服务规则集（DIRECT）
6. 国内 IP（GEOIP CN → DIRECT）
7. 兜底规则（MATCH → PROXY）
```

### 规则集使用
- 优先使用规则仓库中的规则集
- 避免重复创建相同规则集
- 使用开源维护的规则集（如 Loyalsoldier）

### 规则优化
- 常用规则放在前面
- 避免过于宽泛的规则影响性能
- 定期审查规则有效性
