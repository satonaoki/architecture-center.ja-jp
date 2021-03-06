---
title: "破損対策レイヤー パターン"
description: "最新アプリケーションとレガシ システムの間にファサード、すなわちアダプター レイヤーを実装します。"
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: e41f080abbef772596ee7f8b10ad72bb03a3b829
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2018
---
# <a name="anti-corruption-layer-pattern"></a>破損対策レイヤー パターン

最新アプリケーションと、そのアプリケーションが依存するレガシ システムの間にファサード、すなわちアダプター レイヤーを実装します。 このレイヤーにより、最新アプリケーションとレガシ システムの間の要求が変換されます。 このパターンを使用して、アプリケーションの設計が、レガシ システムへの依存関係で制限されないようにします。 このパターンは、Eric Evans が『*Domain-Driven Design*』で初めて説明しました。

## <a name="context-and-problem"></a>コンテキストと問題

アプリケーションのほとんどが、そのデータや機能の一部について、他のシステムに依存しています。 たとえば、レガシ アプリケーションを最新のシステムに移行するとき、そのレガシ アプリケーションは、引き続き既存のレガシ リソースを必要とする可能性があります。 レガシ システムは、新しい機能で呼び出すことができなければなりません。 これは、特に、段階的な移行の場合に当てはまります。こうした移行では、大規模なアプリケーションのさまざまな機能が、時間をかけて最新システムに移動します。

このようなレガシ システムには、多くの場合、データ スキーマが複雑、API が古いなど、品質に関する問題があります。 レガシ システムで使用されている機能とテクノロジは、最新のシステムとは大きく異なる可能性があります。 レガシ システムと相互運用するには、新しいアプリケーションには、最新アプリケーションに組み込まない古いインフラストラクチャ、プロトコル、データ モデル、API、またはその他の機能のサポートが必要になることがあります。

新しいシステムとレガシ システムの間のアクセスを維持することによって、新しいシステムは、少なくともレガシ システムの API または他のセマンティクスの一部に強制的に従うことになります。 こうしたレガシ機能の品質に問題がある場合、その機能をサポートすることで、正常に設計された最新のアプリケーションが "破損" する場合があります。 

## <a name="solution"></a>解決策

レガシ システムと最新システムの間に破損対策レイヤーを配置し、これらを切り離します。 このレイヤーは、2 つのシステム間の通信を変換するもので、レガシ システムはそのままで、最新のアプリケーションの設計と技術的アプローチも損なわれません。

![](./_images/anti-corruption-layer.png) 

最新のアプリケーションと破損対策レイヤーの間の通信では、常に、アプリケーションのデータ モデルとアーキテクチャが使用されます。 破損対策レイヤーからレガシ システムへの呼び出しは、そのシステムのデータ モデルまたはメソッドに準拠します。 破損対策レイヤーには、2 つのシステム間の変換に必要なロジックがすべて含まれます。 レイヤーは、アプリケーション内でコンポーネントとして実装できます。また、独立したサービスとして実装することもできます。

## <a name="issues-and-considerations"></a>問題と注意事項

- 破損対策レイヤーにより、2 つのシステム間で行われた呼び出しで、待ち時間が追加される場合があります。
- 破損対策レイヤーにより、管理および保守しなければならない追加サービスが増えます。
- 破損対策レイヤーをスケールする方法を検討してください。
- 複数の破損対策レイヤーが必要かどうかを検討してください。 さまざまなテクノロジや言語を使用して、機能を複数のサービスに分解できます。破損対策レイヤーをパーティション分割する理由が他に存在する可能性もあります。
- 他のアプリケーションまたはサービスとの関係で、破損対策レイヤーがどのように管理されるかを考慮してください。 これは、監視、リリース、および構成プロセスにどのように統合されますか。
- トランザクションおよびデータの整合性は保持され、監視できることを確認してください。
- 破損対策レイヤーが処理する必要があるのが、レガシ システムと最新システムの間のすべての通信か、または機能のサブセットのみかを検討してください。 
- 破損対策レイヤーを永続的に使うか、いずれ、すべてのレガシ機能が移行された時点で使用をやめるのかを検討します。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の状況で使用します。

- 複数の段階にわたって移行が発生するが、新しいシステムとレガシ システムの間の統合を保持する必要があるとき。
- 新しいシステムとレガシ システムでセマンティクスが異なるが、通信する必要があるとき。

新しいシステムとレガシ システムの間で大きなセマンティクスの違いがない場合、このパターンは適切でない可能性があります。 

## <a name="related-guidance"></a>関連するガイダンス

- [ストラングラー パターン][strangler]

[strangler]: ./strangler.md
