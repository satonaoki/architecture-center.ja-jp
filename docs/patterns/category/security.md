---
title: "セキュリティのパターン"
description: "セキュリティとは、設計された用途以外の悪意あるアクションや偶発的なアクションを防ぎ、情報の漏えいや損失を防ぐシステムの機能です。 クラウド アプリケーションは、信頼できるオンプレミス境界の外側でインターネット上に存在します。一般に公開されていることも多く、信頼できないユーザーが利用する場合もあります。 アプリケーションの設計とデプロイでは、悪意のある攻撃から保護し、承認したユーザーのみにアクセスを制限し、機密データを保護する必要があります。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 266b5c4283d82a107783fc7a746f065be9027b51
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="security-patterns"></a>セキュリティのパターン

[!INCLUDE [header](../../_includes/header.md)]

セキュリティとは、設計された用途以外の悪意あるアクションや偶発的なアクションを防ぎ、情報の漏えいや損失を防ぐシステムの機能です。 クラウド アプリケーションは、信頼できるオンプレミス境界の外側でインターネット上に存在します。一般に公開されていることも多く、信頼できないユーザーが利用する場合もあります。 アプリケーションの設計とデプロイでは、悪意のある攻撃から保護し、承認したユーザーのみにアクセスを制限し、機密データを保護する必要があります。

| パターン | 概要 |
| ------- | ------- |
| [フェデレーション ID](../federated-identity.md) | 外部の ID プロバイダーに認証を委任します。 |
| [ゲートキーパー](../gatekeeper.md) | 専用のホスト インスタンスを使用して、アプリケーションとサービスを保護します。このホスト インスタンスは、クライアントと、アプリケーションまたはサービスとの間でブローカーとして機能し、要求を検証して不適切な部分を除去します。その後、クライアントと、アプリケーションまたはサービスとの間で要求とデータを渡します。 |
| [バレット キー](../valet-key.md) | 特定のリソースまたはサービスへの限定的な直接アクセスをクライアントに提供する、トークンまたはキーを使用します。 |