---
title: "ガイダンス: Azure リソース グループ設計"
description: "基本のクラウド導入戦略の一環である Azure リソース グループ設計のガイダンス"
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-resource-group-design"></a>ガイダンス: Azure リソース グループ設計

Azure では、[リソース グループ](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups)とは、内部のリソースがグループ化されている論理コンテナーです。 Azure にデプロイされた各リソースは、単一のリソース グループにデプロイされる必要があります。

## <a name="design-considerations"></a>設計上の考慮事項

- リソース グループ内のすべてのリソースで、同じライフサイクルを共有する必要がある。 つまり、グループとして同じライフ サイクルで、リソースをデプロイ、更新、削除する手法にする必要があります。 たとえば、単一の Web アプリケーションのコンピューティング リソースは通常、単一ユニットとしてデプロイされます。 ただし、他の Web アプリケーションで共有しているデータベースはほとんどの場合、別のライフサイクルで管理されており、独自のリソース グループ内に配置されています。
- リソース グループには、別のリージョンに存在するリソースを含めることができる。
- 単一のリソース グループ内のすべてのリソースは、1 つのサブスクリプションに関連付ける必要がある。 
- リソースはリソース グループ間で移動できるが、異なるサブスクリプションにデプロイされているリソースを含むリソース グループには移動できない。
- リソースのグループ割り当ては、他のリソース グループにあるリソースとの接続性または対話性に影響を及ぼさない。 たとえば、1 つのリソース グループに割り当てられている仮想マシンは、両者間にネットワーク接続がある場合は、別のリソース グループに割り当てられているデータベースに接続できます。
- リソース グループを使用すると、管理操作のアクセス制御のスコープを設定できる。 サブスクリプション レベルまたはリソース グループ レベルで、ロールベースのアクセス制御 (RBAC) を行うアクセス許可を適用することができます。 サブスクリプション レベルで割り当てられたアクセス許可はすべて、リソース グループ レベルに継承されます。 RBAC とリソースへのアクセス許可については、中間および高度導入ステージで詳しく説明します。

## <a name="proven-practices"></a>実証済みプラクティス

- 基本ステージでは、数個の概念実証 (POC) プロジェクトのみを管理しており、各プロジェクトのリソース数も少量でした。 POC リソースは通常、同じライフ サイクルを共有するので、これらの各プロジェクトに対して 1 つのリソース グループを作成できます。
- 中間導入ステージでは、複数のプロジェクトを管理することになります。 異なる種類のプロジェクトでは、他のリソース グループ設計が有益な場合があります。 任意の初期 POC プロジェクトを運用環境に昇格することを検討している場合、同じサブスクリプションに属していれば、リソースを別のリソース グループに移動できます。 そのため、このステージでは、将来リソースを再編成できように、これらのリソースを同じサブスクリプションにデプロイしてください。

## <a name="next-steps"></a>次の手順

* 基本導入ステージの実証済みプラクティスについて学習したので、リソース グループを作成して、リソースを追加できます。 このステージでは少数のリソースを管理していますが、追加するリソース数が多くなるにつれて、リソースの管理はより複雑になります。 [Azure の名前規則とタグ付け](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)について学習し、中間導入ステージへの準備でリソースへの名前付けとタグ付けを行ってください。