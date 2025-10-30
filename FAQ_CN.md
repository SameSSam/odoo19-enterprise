# 常见问题解答 (FAQ)

## 📌 企业模块相关问题

### 1. 这个Odoo企业版的模块需要额外下载吗？

**是的，需要额外下载。** Odoo 19 企业版模块不包含在本仓库中，需要从Odoo官方的企业版仓库下载。

#### 为什么需要单独下载？

- **许可证限制**：Odoo企业版模块受Odoo Enterprise Edition License保护，不能直接公开分发
- **访问控制**：需要有效的GitHub个人访问令牌(Personal Access Token)才能访问官方企业版仓库
- **版权保护**：Odoo SA保留企业版模块的版权，需要通过授权方式获取

#### 如何下载企业模块？

企业模块会在部署过程中自动下载。本项目的 `deploy.sh` 脚本会：

1. **自动克隆**：使用您提供的GitHub令牌从官方仓库克隆企业模块
2. **自动配置**：将企业模块目录正确配置到Odoo的addons路径中
3. **自动验证**：检查下载的模块数量(应该有700+个模块)

```bash
# 部署时会自动执行以下操作
git clone -b 19.0 https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/odoo/enterprise.git enterprise
```

#### 手动下载企业模块的步骤：

如果需要手动下载，可以执行以下步骤：

```bash
# 进入安装目录
cd /opt/odoo19-enterprise

# 克隆企业版仓库 (需要有效的GitHub令牌)
git clone -b 19.0 https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/odoo/enterprise.git enterprise

# 验证模块数量
ls -1 enterprise | wc -l
# 应该显示 700+ 个模块

# 重启Odoo以加载新模块
./manage.sh restart
```

---

### 2. 如果/enterprise下没有某个模块显示，Update Module会从哪里更新模块？

#### 模块更新来源

**Update Module功能会从Odoo官方GitHub企业版仓库更新模块**

- **官方仓库**：`https://github.com/odoo/enterprise.git`
- **分支**：`19.0` (Odoo 19版本)
- **更新方式**：通过Git Pull获取最新的模块代码

#### 更新过程详解

当您执行模块更新时（使用 `./update.sh modules`），系统会：

1. **连接到官方仓库**
   ```bash
   cd enterprise
   git fetch origin
   ```

2. **拉取最新更改**
   ```bash
   git pull origin 19.0
   ```

3. **验证更新**
   - 检查拉取的模块数量
   - 显示最近的提交记录
   - 确保所有模块正确更新

4. **重启Odoo**
   ```bash
   docker compose restart odoo19
   ```

5. **刷新应用列表**
   - 登录Odoo
   - 进入"应用"菜单
   - 点击"更新应用列表"按钮
   - 新模块和更新将出现在列表中

#### 如何手动更新模块？

使用提供的更新脚本：

```bash
# 方法1：仅更新企业模块
cd /opt/odoo19-enterprise
./update.sh modules

# 方法2：更新所有组件（模块+容器）
./update.sh all

# 方法3：检查可用更新
./update.sh check
```

#### 如果模块没有显示怎么办？

如果某个模块在 `/enterprise` 目录下不存在或不显示，请按照以下步骤排查：

**步骤1：检查enterprise目录**
```bash
# 检查目录是否存在
ls -la /opt/odoo19-enterprise/enterprise

# 检查模块数量
ls -1 /opt/odoo19-enterprise/enterprise | wc -l

# 搜索特定模块（例如：studio）
ls -la /opt/odoo19-enterprise/enterprise | grep studio
```

**步骤2：检查Git仓库状态**
```bash
cd /opt/odoo19-enterprise/enterprise
git status
git remote -v
git branch
```

**步骤3：强制更新企业模块**
```bash
cd /opt/odoo19-enterprise/enterprise
git fetch origin
git reset --hard origin/19.0
cd ..
./manage.sh restart
```

**步骤4：在Odoo中刷新应用列表**
1. 登录Odoo系统
2. 进入"应用"菜单
3. 移除所有筛选器
4. 点击右上角的"更新应用列表"按钮
5. 在弹出的对话框中点击"更新"
6. 等待更新完成后，搜索需要的模块

**步骤5：检查Odoo配置**
```bash
# 检查addons路径配置
cat /opt/odoo19-enterprise/config/odoo.conf | grep addons_path

# 应该包含：
# addons_path = /mnt/enterprise-addons,/usr/lib/python3/dist-packages/odoo/addons
```

---

### 3. GitHub个人访问令牌(Personal Access Token)是什么？

#### 令牌的作用

**GitHub个人访问令牌是下载企业版模块的凭证**

个人访问令牌(Personal Access Token, PAT)用于：
- ✅ 验证您有权限访问Odoo企业版仓库
- ✅ 在自动化脚本中安全地进行身份验证
- ✅ 替代密码进行Git操作，更加安全
- ✅ 可以设置特定权限和过期时间

#### 如何创建GitHub个人访问令牌？

**步骤1：登录GitHub**
1. 访问 https://github.com
2. 使用您的GitHub账号登录

**步骤2：进入令牌设置页面**
1. 点击右上角头像
2. 选择 **Settings** (设置)
3. 在左侧菜单底部找到 **Developer settings** (开发者设置)
4. 点击 **Personal access tokens** → **Tokens (classic)**

**步骤3：生成新令牌**
1. 点击 **Generate new token (classic)** 按钮
2. 输入令牌描述，例如："Odoo Enterprise Module Access"
3. 选择过期时间（建议选择 **No expiration** 或较长期限）

**步骤4：选择权限范围**

必须选择以下权限：
- ✅ **repo** (完整的私有仓库控制权限)
  - 包含 repo:status, repo_deployment, public_repo, repo:invite 等子权限
- ✅ **read:org** (读取组织和团队成员信息)

**步骤5：生成并保存令牌**
1. 滚动到页面底部，点击 **Generate token** 按钮
2. **立即复制令牌！** 令牌格式类似：`ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
3. ⚠️ **重要**：令牌只显示一次，请妥善保存
4. 建议保存到密码管理器中

#### 如何使用令牌？

**在部署时使用令牌：**

```bash
# 交互式部署
./deploy.sh
# 当提示时输入：
# GitHub Username: your_username
# GitHub Token: ghp_your_token_here

# 自动化部署
./deploy.sh --auto \
  --github-user "your_username" \
  --github-token "ghp_your_token_here" \
  --main-port 10024 \
  --longpolling-port 20024
```

**使用环境变量：**

```bash
# 设置环境变量
export GITHUB_USER="your_username"
export GITHUB_TOKEN="ghp_your_token_here"

# 使用环境变量部署
./deploy.sh --env
```

#### 令牌安全注意事项

- 🔒 **不要公开分享**：永远不要将令牌提交到公共仓库
- 🔒 **定期更换**：建议定期更换令牌以提高安全性
- 🔒 **最小权限原则**：只授予必要的权限
- 🔒 **妥善保管**：使用密码管理器保存令牌
- 🔒 **发现泄露立即撤销**：如果令牌泄露，立即在GitHub上撤销

#### 令牌相关问题

**Q: 令牌过期了怎么办？**
- 在GitHub上生成新的令牌
- 使用新令牌更新部署配置

**Q: 如何验证令牌是否有效？**
```bash
# 测试令牌
curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user

# 测试仓库访问权限
git ls-remote https://YOUR_USERNAME:YOUR_TOKEN@github.com/odoo/enterprise.git
```

**Q: 令牌丢失了怎么办？**
- 令牌无法恢复，只能重新生成新的令牌
- 撤销旧令牌，生成新令牌并更新配置

---

### 4. 需要什么权限才能访问企业模块？

#### 访问要求

要下载和使用Odoo企业版模块，您需要：

**1. Odoo企业版授权**
   - Odoo官方合作伙伴身份
   - 或有效的Odoo企业版订阅
   - 或Odoo员工账号

**2. GitHub账号**
   - 普通的GitHub账号
   - 需要被添加到Odoo企业版仓库的访问列表中

**3. GitHub个人访问令牌**
   - 如上所述创建的PAT
   - 包含必要的权限范围

#### 如何获取访问权限？

**方法1：通过Odoo合作伙伴计划**
1. 注册成为Odoo官方合作伙伴
2. 联系Odoo支持团队
3. 提供您的GitHub用户名
4. Odoo会将您的账号添加到企业版仓库

**方法2：通过企业版订阅**
1. 购买Odoo企业版订阅
2. 在Odoo.com账户中关联GitHub账号
3. 自动获得仓库访问权限

**方法3：联系Odoo支持**
- 邮件：support@odoo.com
- 网站：https://www.odoo.com/help

---

### 5. 支持哪些企业模块？

#### 主要企业模块类别

本部署包含 **719+** 个企业模块，涵盖以下主要功能：

**🎨 开发与定制**
- Odoo Studio - 可视化应用构建器
- Web Enterprise - 企业版界面主题

**📁 文档管理**
- Documents - 文档管理系统
- Documents Spreadsheet - 电子表格集成
- Sign - 电子签名

**🎫 客户服务**
- Helpdesk - 高级工单系统
- Live Chat - 在线客服
- Knowledge - 知识库管理

**📅 项目与资源管理**
- Planning - 资源规划
- Project Enterprise - 企业版项目管理
- Timesheet Grid - 工时表视图

**💼 财务与会计**
- Accounting Enterprise - 高级会计功能
- Account Reports - 会计报表
- Account Consolidation - 账目合并
- Account Sepa - SEPA支付

**👥 人力资源**
- HR Payroll - 工资管理
- HR Appraisal - 绩效评估
- HR Referral - 员工推荐
- HR Skills - 技能管理

**🏭 制造与库存**
- MRP - 制造资源规划
- Quality - 质量控制
- PLM - 产品生命周期管理
- Maintenance - 设备维护

**📈 营销与销售**
- Marketing Automation - 营销自动化
- Social Marketing - 社交媒体营销
- Email Marketing - 邮件营销
- Sale Enterprise - 企业版销售

**🔧 现场服务**
- Field Service - 现场服务管理
- Industry FSM - 行业现场服务

**📱 移动应用**
- Mobile Apps - iOS/Android应用支持
- Web Mobile - 移动Web界面

**🌐 电子商务**
- Website Enterprise - 企业版网站
- eCommerce - 电子商务

**📊 商业智能**
- Dashboards - 仪表板
- Spreadsheet - 电子表格
- Data Cleaning - 数据清理

#### 查看所有可用模块

```bash
# 列出所有企业模块
ls -1 /opt/odoo19-enterprise/enterprise

# 查看模块详细信息
cat /opt/odoo19-enterprise/enterprise/MODULE_NAME/__manifest__.py

# 统计模块数量
ls -1 /opt/odoo19-enterprise/enterprise | wc -l
```

---

### 6. 模块更新的最佳实践

#### 更新前的准备

**1. 创建完整备份**
```bash
cd /opt/odoo19-enterprise
./backup.sh create
```

**2. 检查可用更新**
```bash
./update.sh check
```

**3. 在测试环境先更新**
- 如果可能，先在测试或开发环境更新
- 验证更新不会破坏现有功能

#### 执行更新

**更新企业模块**
```bash
./update.sh modules
```

**更新所有组件**
```bash
./update.sh all
```

#### 更新后的验证

**1. 检查服务状态**
```bash
./manage.sh status
docker compose ps
```

**2. 查看日志**
```bash
./manage.sh logs
```

**3. 在Odoo中验证**
- 登录Odoo
- 进入"应用"菜单
- 点击"更新应用列表"
- 检查模块是否正常工作

**4. 测试关键功能**
- 测试已安装的模块
- 验证数据完整性
- 检查自定义功能

#### 回滚更新

如果更新出现问题：

**回滚模块更新**
```bash
./update.sh rollback modules
```

**恢复完整备份**
```bash
./backup.sh list
./backup.sh restore backup_YYYYMMDD_HHMMSS.tar.gz
```

---

## 🔧 故障排查

### 企业模块无法下载

**问题症状**：
- 部署时提示无法克隆enterprise仓库
- 提示认证失败

**解决方案**：

1. **验证GitHub令牌**
   ```bash
   # 测试令牌是否有效
   curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user
   ```

2. **检查令牌权限**
   - 确保令牌包含 `repo` 权限
   - 令牌未过期

3. **验证仓库访问权限**
   ```bash
   # 测试是否能访问企业仓库
   git ls-remote https://YOUR_USERNAME:YOUR_TOKEN@github.com/odoo/enterprise.git
   ```

4. **更新令牌**
   - 生成新的GitHub令牌
   - 使用新令牌重新部署

### 模块更新失败

**问题症状**：
- `./update.sh modules` 执行失败
- Git pull报错

**解决方案**：

1. **检查网络连接**
   ```bash
   ping github.com
   curl -I https://github.com
   ```

2. **检查Git配置**
   ```bash
   cd /opt/odoo19-enterprise/enterprise
   git config --list
   git remote -v
   ```

3. **清理本地更改**
   ```bash
   cd /opt/odoo19-enterprise/enterprise
   git status
   git stash  # 如果有本地修改
   git pull origin 19.0
   ```

4. **强制重置到远程版本**
   ```bash
   cd /opt/odoo19-enterprise/enterprise
   git fetch origin
   git reset --hard origin/19.0
   ```

### 模块在Odoo中不显示

**问题症状**：
- 企业模块已下载但在Odoo应用列表中找不到

**解决方案**：

1. **检查addons路径**
   ```bash
   docker compose exec odoo19 cat /etc/odoo/odoo.conf | grep addons_path
   ```

2. **验证模块目录挂载**
   ```bash
   docker compose exec odoo19 ls -la /mnt/enterprise-addons
   ```

3. **重启Odoo并更新应用列表**
   ```bash
   ./manage.sh restart
   ```
   然后在Odoo界面：
   - 进入"应用"菜单
   - 移除所有筛选器
   - 点击"更新应用列表"

4. **检查模块依赖**
   - 某些企业模块可能依赖于其他模块
   - 先安装依赖模块

5. **查看Odoo日志**
   ```bash
   ./manage.sh logs | grep -i "module\|addon"
   ```

### 容器无法访问企业模块目录

**问题症状**：
- Odoo日志显示无法找到企业模块路径
- 模块加载失败

**解决方案**：

1. **检查Docker卷挂载**
   ```bash
   docker compose config | grep volumes -A 10
   ```

2. **验证目录权限**
   ```bash
   ls -la /opt/odoo19-enterprise/enterprise
   chmod -R 755 /opt/odoo19-enterprise/enterprise
   ```

3. **重新创建容器**
   ```bash
   docker compose down
   docker compose up -d
   ```

---

## 📞 获取帮助

### 文档资源

- **安装指南**：[INSTALL.md](INSTALL.md)
- **英文README**：[README.md](README.md)
- **配置文件示例**：`.env.example`

### 社区支持

- **GitHub Issues**：报告问题和bug
- **GitHub Discussions**：讨论和问答
- **Odoo官方文档**：https://www.odoo.com/documentation/19.0/

### 常用命令速查

```bash
# 服务管理
./manage.sh start          # 启动服务
./manage.sh stop           # 停止服务
./manage.sh restart        # 重启服务
./manage.sh status         # 查看状态
./manage.sh logs           # 查看日志

# 备份管理
./backup.sh create         # 创建备份
./backup.sh list          # 列出备份
./backup.sh restore <file> # 恢复备份

# 更新管理
./update.sh modules        # 更新企业模块
./update.sh odoo          # 更新Odoo容器
./update.sh all           # 更新所有组件
./update.sh check         # 检查更新
```

---

## 📝 附加说明

### 许可证信息

- **本部署脚本**：MIT License
- **Odoo社区版**：LGPL-3.0 License
- **Odoo企业版模块**：Odoo Enterprise Edition License (OEEL-1)
  - 需要有效的Odoo企业版订阅
  - 不能公开分发或转售

### 版本兼容性

- **Odoo版本**：19.0
- **PostgreSQL版本**：17
- **Python版本**：3.10+
- **Docker版本**：20.10+
- **Docker Compose版本**：2.0+

### 系统要求

**最低配置**：
- CPU: 2核
- 内存: 4GB
- 磁盘: 20GB
- 网络: 稳定的互联网连接

**推荐配置**：
- CPU: 4核或更多
- 内存: 8GB或更多
- 磁盘: 50GB SSD
- 网络: 高速互联网连接

**生产环境配置**：
- CPU: 8核或更多
- 内存: 16GB或更多
- 磁盘: 100GB+ SSD
- 网络: 1Gbps或更快
- 自动备份方案

---

**更新日期**：2025年10月  
**文档版本**：1.0.0  
**适用于**：Odoo 19 Enterprise Deployment Repository
