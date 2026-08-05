---
title: Cloudflare 钱包上线：为代理经济构建支付与身份基础设施
description: Cloudflare 钱包通过账户钱包、虚拟钱包和 cloudflare.pay 句柄，为 AI 代理提供稳定的身份标识与原生支付能力，旨在降低代理调用 API 的门槛，推动机器间自主交易的发展。
author: 小狐
cover: https://bed.foxmoe.top/file/images/1785896741228_image.png
coverAlt: Cloudflare 钱包上线：为代理经济构建支付与身份基础设施
pubDate: 2026-08-05
updatedDate: 2026-08-05
tags: [Cloudflare,钱包,AI代理,微支付,稳定币,身份标识,代理经济]
categories: [技术新闻]
featured: false
toc: true
---

2026 年 8 月 4 日，Cloudflare 官方博客发布了题为《Announcing Cloudflare Wallets: the programmable wallet for the agentic Internet》的公告，正式推出 Cloudflare 钱包服务。

**地址**：[https://cloudflare.pay/](https://cloudflare.pay/)  

---
## Cloudflare 钱包：先认领标识，支付功能后续开放

2026 年 8 月 4 日，Cloudflare 宣布推出 Cloudflare Wallets。目前开放的第一项能力是账户标识认领，完整的充值、付款和收款功能将在后续逐步提供。

这一点需要首先明确：用户可以前往 cloudflare.pay 为账户认领一个唯一的 Wallet handle（即容易记忆的账户标识），但还不能把它当作已经全面开放的稳定币钱包来使用。Cloudflare 现阶段开放的是标识绑定，而非完整支付能力。

### 为什么需要这个钱包

Cloudflare Wallets 面向的是需要在网络上自动购买服务的 AI Agent。当前，AI Agent 调用 API 时面临两个核心障碍：没有稳定的身份标识供服务商识别，也没有原生的支付手段。注册、绑定支付方式、生成 API Key 等环节都需要人工介入，Agent 无法独立完成完整的购买流程。

Cloudflare 的做法是把身份和支付两件事打包进一个钱包体系。

### 两类钱包：Account Wallets 与 Virtual Wallets

Wallets 计划分为两类：

- **Account Wallets**：由 Cloudflare 账户所有者管理，用于存入资金、取出资金，并把部分支出权限授权给虚拟钱包。
- **Virtual Wallets**：面向 AI Agent，通过 API 密钥操作。一个账户可以为不同 Agent 创建多个虚拟钱包，让它们购买 API、MCP 工具、内容等网络资源。MCP 是一套让 AI 调用外部工具和数据的通用接口。

资金仍由账户所有者控制。每个虚拟钱包可以设置可用额度、允许交易的对象以及单笔交易上限。超出限制后，Agent 需要请求有权限的人调整规则或批准额外资金。这种设计让 Agent 不必在每一笔小额购买前等待人工确认，同时为自动支出保留了明确边界。

### x402 协议：把付款附在请求里

Cloudflare Wallets 计划通过 x402 协议购买服务。x402 把 HTTP 的 `402 Payment Required` 状态码用于机器间支付：服务端返回价格和收款要求，调用方签署付款信息后再次发起请求，验证和结算完成后获取资源。

它适合金额小、调用频率高的场景。例如，Agent 可以按次购买一次 API 调用、一个数据集查询或一次 MCP 工具执行，不必先注册每家服务、绑定银行卡并购买整包额度。

Wallets 对应买方一侧。Cloudflare 在 7 月推出的 Monetization Gateway 对应卖方一侧，让网站、API 和 MCP 工具提供者通过 x402 对资源收费。两项能力组合后，卖方可以设置收费规则，Agent 从受限的虚拟钱包完成支付。

### Wallet handle 同时承担身份作用

钱包标识不只用于付款。Cloudflare 计划让 Agent 使用类似 `research.example.cloudflare.pay` 的地址声明身份。这个可读地址对应 Agent 的密钥，让商家能够识别它受哪个账户委托。

是否公开身份由 Agent 一方选择，商家也可以自行决定是否优先与已识别的 Agent 交易。Cloudflare 没有为身份附加一套固定资料格式，而是先把难以阅读的密钥转换成容易记忆和传递的名称。

### 当前状态与后续计划

对于普通用户，目前可执行的操作只有认领 Wallet handle。后续支付能力开放后，账户所有者才可以添加资金、创建虚拟钱包并设置支出规则。

认领地址：[https://cloudflare.pay](https://cloudflare.pay)

---