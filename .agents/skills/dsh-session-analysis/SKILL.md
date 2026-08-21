# dsh 会话分析与沉淀技能

## 目标

在量潮领域工作区用 dsh（DeepSeek Harness）开对话（含视觉模型分析网页 UI），把分析结果沉淀到本 context 仓库。

## 流程

### 第一步：确认工作区

```bash
ls ~/.dsh/sessions/ | grep <领域名>
cd /home/iguo/repos/quanttide/domains/quanttide-<领域名>
```

### 第二步：dsh 对话

在领域仓库根目录运行（cwd 即工作区）：

```bash
dsh --profile headless "任务描述（中文，明确要分析什么、输出什么）"
```

### 第三步：视觉模型分析网页 UI（进阶）

```bash
# 1. 截图（chrome headless）
google-chrome --headless=new --disable-gpu --no-sandbox \
  --window-size=1440,900 --screenshot=/tmp/ui-shot/full.png \
  --virtual-time-budget=15000 "https://<域名>/"

# 2. 临时切换视觉模型（~/.dsh/settings.yaml）
sed -i 's|model: deepseek-v4-flash$|model: deepseek-v4-flash-vision-exp|' ~/.dsh/settings.yaml

# 3. 视觉分析
dsh --profile headless "分析图片 /tmp/ui-shot/full.png 的 UI 和 UX 设计优缺点"

# 4. 恢复默认模型
sed -i 's|model: deepseek-v4-flash-vision-exp$|model: deepseek-v4-flash|' ~/.dsh/settings.yaml
```

### 第四步：沉淀到 context

- 结构分析类：`<主题>.md`（如 `design-domain-structure.md`）
- UI/UX 分析类：`<app-name>/ui-ux-analysis-<日期>.md`（如 `qtcloud-product-studio/`）
- 头部标注来源：`> 来源：dsh headless 会话分析（YYYY-MM-DD）`
- 内容提炼核心，不全文照搬

### 第五步：分层提交推送

```bash
cd data/context
git add <文件>
git commit -m "docs: <内容>（dsh 会话沉淀）"
git push
cd ../..
git add data/context
git commit -m "chore: update context submodule"
git push
```

## 验收标准

- [ ] dsh 对话在目标工作区成功运行（有实质输出）
- [ ] 视觉分析时：截图完整、模型临时切换并已恢复默认
- [ ] 结果写入本仓库（带来源标注，路径符合分类规范）
- [ ] 分层提交推送（context → 领域父仓库），指针同步

## 注意事项

1. dsh 会话基于工作目录（cwd）——必须在领域仓库根运行
2. 视觉模型临时切换后**必须恢复** deepseek-v4-flash（默认）
3. 超时：复杂分析任务 `timeout 300`；截图 `--virtual-time-budget=15000`（等 SPA 渲染）
4. UI/UX 分析结果直接可作产品迭代输入（能指出具体文案 bug、层级问题）
