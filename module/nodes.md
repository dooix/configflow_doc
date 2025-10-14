# 节点管理

节点管理模块提供了完整的节点增删改查功能，支持单个添加和批量导入，是功能最全面的模块之一。

## 主要功能

### 1. 节点列表展示

节点以卡片形式展示，每个节点卡片包含：
- **节点名称**：易于识别的节点名称
- **协议类型**：自动识别并显示协议标签（SS、VMess、Trojan等）
- **节点字符串**：格式化显示节点配置（支持 URI、JSON、YAML格式）
- **来源订阅**：如果节点来自订阅，会显示订阅名称标签
- **启用状态**：通过开关控制节点是否启用

### 2. 支持的协议类型

- **SS (Shadowsocks)**：标签颜色为蓝色
- **VMess**：标签颜色为绿色
- **VLess**：标签颜色为绿色
- **Trojan**：标签颜色为橙色
- **Hysteria2**：标签颜色为红色
- **HTTP/HTTPS**：标签颜色为灰色/浅蓝色

### 3. 单个节点添加

支持三种格式输入：

**格式1：URI 格式（推荐）**
```
vmess://eyJhZGQiOiJoay5leGFtcGxlLmNvbSIsImFpZCI6IjAi...
vless://7ab34cf6-1a99-4168-9669-53c5f116eb29@hk.example.com:443...
ss://YWVzLTI1Ni1nY206cGFzc3dvcmQ=@hk.example.com:8388#香港节点
trojan://password@us.example.com:443#美国节点
hysteria2://password@jp.example.com:443#日本节点
```

**格式2：JSON 格式（单行对象）**
```json
{name: 香港节点, type: ss, server: hk.example.com, port: 8388, cipher: aes-256-gcm, password: pass123}
```

**格式3：YAML 格式（多行）**
```yaml
name: 香港节点
type: vless
server: hk.example.com
port: 443
uuid: 7ab34cf6-1a99-4168-9669-53c5f116eb29
encryption: none
security: reality
sni: addons.mozilla.org
```

### 4. 批量添加节点

批量添加对话框支持：
- **多行输入**：每行一个节点链接
- **混合格式**：可以混合使用 URI、JSON 格式
- **自动提取名称**：从 URI 格式的 `#` 后自动提取节点名称
- **自动编号**：如果无法提取名称，自动命名为"节点_1"、"节点_2"等
- **注释支持**：以 `//` 开头的行会被忽略
- **空行忽略**：自动过滤空行

**批量添加示例：**
```
// 香港节点
vmess://xxxxx#香港01
vmess://xxxxx#香港02

// 美国节点
trojan://password@us1.example.com:443#美国01
trojan://password@us2.example.com:443#美国02
```

### 5. 节点编辑

- 支持修改节点名称
- 支持修改节点字符串配置
- JSON 格式会自动格式化为多行便于编辑
- 修改后自动保存

### 6. 节点删除

**单个删除：**
- 点击删除按钮
- 自动清理策略组中对该节点的引用
- 包括 `manual_nodes` 和 `proxies_order` 中的引用

**批量删除：**
- 通过复选框选择多个节点
- 点击"全选"可快速选择所有节点
- 批量删除按钮显示选中数量
- 一次性删除多个节点并清理所有相关引用
- 显示删除成功数量和清理的策略组数量

### 7. 节点启用/禁用

- 每个节点卡片包含启用开关
- 禁用的节点不会出现在生成的配置中
- 可以快速启用/禁用节点而不删除

### 8. 节点排序

- 拖拽节点卡片左上角的手柄图标
- 调整节点在列表中的顺序
- 顺序会影响策略组中的节点显示顺序
- 排序自动保存

## 使用场景

**场景1：添加单个自建节点**
```yaml
name: 自建香港VPS
type: vless
server: 192.168.1.100
port: 443
uuid: 550e8400-e29b-41d4-a716-446655440000
encryption: none
flow: xtls-rprx-vision
```

**场景2：批量导入机场节点**
```
ss://YWVzLTI1Ni1nY206cGFzc3dvcmQx@hk1.example.com:8388#香港节点01
ss://YWVzLTI1Ni1nY206cGFzc3dvcmQy@hk2.example.com:8388#香港节点02
ss://YWVzLTI1Ni1nY206cGFzc3dvcmQz@us1.example.com:8388#美国节点01
```

**场景3：清理失效节点**
- 使用全选功能选择所有节点
- 取消勾选需要保留的节点
- 批量删除失效节点
- 系统自动清理策略组中的引用

## 最佳实践

### 节点命名规范
- 使用清晰的地区标识：香港01、美国02
- 包含线路信息：IPLC、IEPL、BGP
- 示例：`香港-IPLC-01`、`美国-BGP-03`

### 节点分类
- 按地区组织节点
- 使用策略组管理不同用途的节点
- 游戏节点、流媒体节点分开管理

### 批量操作技巧
- 使用批量添加功能快速导入
- 定期清理失效节点
- 使用全选功能配合批量删除
