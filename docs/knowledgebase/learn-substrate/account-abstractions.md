---
title: 账户摘要
---

Substrate使用多套公私钥对来代表网络的参与者。

## 概述

区块链系统有多种身份的参与者，例如从验证者到普通用户。

例如，Substrate节点使用任命的权益证明（NPoS）算法来选择区块验证者。验证者和任命者可能持有大量的资金，所以Substrate的权益模型引入账户摘要来帮助资金尽可能安全。

这些摘要是：

- 账户
  - 储备秘钥： 储备账户有意愿持有大量的资金。它的私钥应该尽可能安全地存在冷钱包中。
  - 操控者秘钥： 操控账户代表储备账户标志选择，例如支付偏好，但应该只能持有最小量的资金来支付交易费用。应该保证它的私钥安全，因为它影响到验证者设置，但是会稍微经常地用来保证验证者平衡。
- 会话秘钥：会话秘钥是保存在验证者客户端的热秘钥，用来签特定的验证者操作。它们不应该持有资金。 
  
> **注意：** 要在Substrate的NPoS上下文中学习更多关于验证者和任命者的[权益模块](https://substrate.dev/rustdocs/v2.0.0-rc4/pallet_staking/index.html).

## 账户秘钥

一对秘钥对能代表一个账户和操控的资金，就像你在其他区块链中认为的普通账户。在Substrate的[余额模块](https://substrate.dev/rustdocs/v2.0.0-rc4/pallet_balances/index.html)上下文中，这些账户必须有最小数量的资金（“必要存款”）来保证存在于存储空间中。

账户秘钥一般地被定义并且在runtime中具体化。

继续我们的储备和操控者账户，这些账户里的秘钥通过使用意向来区分而不是根本的加密的区别。当创建储备或控制者秘钥时，所有的适用于普通账户的密码学在此也适用。

### 储备秘钥

储备秘钥是定义储备账户的公私钥对。这个账户类似于你不应该频繁发送交易的“储蓄账户”。因此，他的私钥应该得到极度安全保障，例如保护在一个安全或者多层的硬件安全的地方。

因为存储秘钥离线保存，它指派了一个控制者账户通过储备账户的资金来做不花钱决定。它也能指派一个代理账户来代表它投票治理。

### 控制着秘钥

控制者秘钥是定义一个控制者账户的公私钥对。在Substrate的NPoS模型下，控制着秘钥会把一个意向标记为验证或者任命。

控制者秘钥用来设置一些偏好，例如激励对象，在验证者方面来说，设置他们的会话秘钥。控制者账户只需要支付交易手续费，所以它只需要最小的资金。

控制者私钥决对不能从存储账户花费资金。然而，控制者账户采取的行为可能会导致资金削减，所以仍然应该被很好地保护。

## 会话秘钥

会话秘钥是被验证者使用来签发共识相关信息的“热秘钥”。他们不打算用来作为控制资金的账户秘钥来使用并且只能为了它们预期目的来使用。它们能被经常改变；你的控制账户只需要通过一个会话公钥来创建一个证书并且通过外部交易来广播这个证书。会话秘钥也是一般化地定义并且在runtime的中具体化。

为了创建一个会话秘钥，验证者的操作者必须证实秘钥起到代表它们的储备账户和任命者的作用。 为了达到这个目的，它们通过用它们的控制者秘钥签发一个秘钥来创建一个证书。然后，通过链中的一个交易来公布会话证书的形式告诉链，这个秘钥代表它们的控制者秘钥。

Substrate提供了[会话模块](https://substrate.dev/rustdocs/v2.0.0-rc4/pallet_session/index.html)，使得验证者能够管理它们的会话秘钥。

### 强类型包装

你可以声明任意数量的会话秘钥。例如，默认的Substrate节点使用3个会话秘钥BABE、GRANDPA、以及 "I'm Online"。其他链或多或少有一些，根据验证者在链上被期望采取的行为。

这些不同的会话秘钥可能使用相同的密码学，但是在runtime逻辑中用于不同目的。为了防止错误的秘钥用到错误的操作中，强Rust类型封装了这些私钥，使得他们和另一个不兼容，并且确保他们只能用于特定的目的。

### 生成

如果一个会话秘钥被破解了，攻击者会犯slashable行为。会话秘钥应该通过[the `rotate_keys` RPC](https://substrate.dev/rustdocs/v2.0.0-rc4/sc_rpc/author/trait.AuthorApi.html#tymethod.rotate_keys)经常改变（例如每个会话），以此来增强安全性。

## 下一步

### 学习更多

- 学习关于 [Substrate中使用的密码学](../advanced/cryptography).

### 案例

- 参考
  [创建本地网络以及生成秘钥教程](../../tutorials/start-a-private-network/).

### 引用

- 查看[会话秘钥runtime API](https://substrate.dev/rustdocs/v2.0.0-rc4/sp_session/trait.SessionKeys.html)的参考文档.

- 查看默认文档[Substrate节点runtime中的会话秘钥](https://substrate.dev/rustdocs/v2.0.0-rc4/node_runtime/struct.SessionKeys.html).

- 查看用来构建应用专用的强类型加密包[`substrate_application_crypto`](https://substrate.dev/rustdocs/v2.0.0-rc4/sp_application_crypto/index.html)。