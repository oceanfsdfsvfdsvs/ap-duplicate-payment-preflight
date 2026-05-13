# AP 重复付款预检查

[English](README.md) | [简体中文](README.zh-CN.md)

在钱离开银行账户之前，先发现重复供应商付款。

本项目是 [实用 Agent Skills](https://github.com/oceanfsdfsvfdsvs/practical-agent-skills) 集合的一部分。

`ap-duplicate-payment-preflight` 是一个本地优先的 agent skill，面向财务、应付账款、运营和创始人团队。它用于在 ACH、wire、银行卡或支票付款前检查付款批次。

它可以帮助 agent 检查导出的发票/账单文件，发现精确重复和近似重复，并生成 AP 审核人员可以直接处理的 exception report；不需要连接 ERP，也不需要处理凭据。

## 解决什么问题

很多 AP 团队依赖 ERP 的重复付款提醒，但这些提醒通常只能抓到精确重复。真实重复付款常见于：

- 供应商别名或重复 vendor master。
- OCR 导致的发票号变体，例如 `INV-1007`、`INV1007`、`1007`。
- 已付款账单被重新提交。
- 已付款记录和待付款记录发生碰撞。

这个 skill 把 AP 审核规则和确定性本地扫描脚本结合起来，让 agent 能稳定输出付款批次风险报告。

## 为什么不只用普通 Prompt

- 会归一化发票号并比较多种变体。
- 会检查 paid-vs-pending、相同 PO、相同金额、相近日期和供应商别名。
- 会区分 `hold_payment`、`ap_review` 和 allow-with-note。
- 会限制 agent 不去批准、取消或修改真实付款。

## 运行示例

```bash
python3 scripts/ap_duplicate_payment_preflight.py \
  --payments scripts/fixtures/ap_payments.csv \
  --date-window-days 14
```

脚本会把 Markdown 报告打印到 stdout。只有在传入 `--output` 时才会写文件。

## 输入格式

CSV header 会大小写不敏感匹配。推荐字段：

```csv
vendor_name,vendor_id,invoice_number,invoice_date,payment_date,amount,currency,status,po_number
```

JSON 可以是 payment objects 列表，也可以是包含 `payments`、`invoices`、`bills` 或 `rows` 的对象。

## 安装说明

- Codex/OpenAI 风格 agents：直接使用本目录的 `SKILL.md` 和 `agents/openai.yaml`。
- Claude Code：复制 `.claude/skills/ap-duplicate-payment-preflight/SKILL.md`，或复制整个目录。
- OpenClaw：查看 `openclaw/README.md`。

本地验证不需要会计系统凭据、银行数据或网络访问。
