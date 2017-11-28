---
title: "アーキテクチャ スタイル"
description: "クラウド アプリケーションの一般的なアーキテクチャ スタイル"
layout: LandingPage
ms.openlocfilehash: 15a316f9ebf7cfe4e72a6992f264a68abb904819
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="architecture-styles"></a>アーキテクチャ スタイル

"*アーキテクチャ スタイル*" とは、特定の性質を持つアーキテクチャのグループです。 たとえば、[n 層][n-tier]は一般的なアーキテクチャ スタイルです。 最近では、[マイクロサービス アーキテクチャ][microservices]がよく使用されるようになりました。 アーキテクチャ スタイルによって特定のテクノロジの使用が必要になることはありませんが、一部のテクノロジは特定のアーキテクチャに適しています。 たとえば、コンテナーはマイクロサービスにとって最適なソリューションです。  

クラウド アプリケーションで一般的に見られる一連のアーキテクチャ スタイルを特定しました。 各スタイルの説明には次の内容が含まれています。

- スタイルの説明と論理図。
- スタイルの選択を推奨するケース。
- 利点、課題、およびベスト プラクティス。
- 関連する Azure サービスを使用する推奨デプロイ。


## <a name="a-quick-tour-of-the-styles"></a>スタイルのクイック ツアー   

このセクションでは、特定したアーキテクチャ スタイルのクイック ツアーを行い、それぞれを使用する場合の考慮事項の要旨も説明します。 詳しくはリンク先のトピックをご覧ください。

### <a name="n-tier"></a>n 層

<img src="./images/n-tier-sketch.svg" style="float:left; margin-top:6px;"/>

**[n 層][n-tier]**は、エンタープライズ アプリケーション用の従来のアーキテクチャです。 論理的な機能 (プレゼンテーション、ビジネス ロジック、データ アクセスなど) を実行する "*レイヤー*" にアプリケーションを分割して依存関係を管理します。 レイヤーが呼び出すことができるのは、その下に配置されているレイヤーのみです。 ただし、この水平レイヤーがデメリットになることもあります。 アプリケーションの一部に変更を加えるときに、その他の部分を変更せずに行うことが難しい場合があるためです。 これにより、頻繁な更新が困難になり、新機能の迅速な追加が妨げられます。

n 層は、レイヤー アーキテクチャを既に使用している既存アプリケーションの移行に最適です。 この理由から、n 層は、サービスとしてのインフラストラクチャ (IaaS) ソリューション、または IaaS と管理対象サービスの両方を使用するアプリケーションで最もよく見られます。 

### <a name="web-queue-worker"></a>Web キューワーカー

<img src="./images/web-queue-worker-sketch.svg" style="float:left; margin-top:6px;"/>

純粋な PaaS ソリューションの場合は、**[Web キューワーカー](./web-queue-worker.md)** アーキテクチャを検討してください。 このスタイルでは、アプリケーションには HTTP 要求を処理する Web フロント エンドと、CPU 集中型タスクまたは実行時間の長い操作を行うバックエンド ワーカーがあります。 フロント エンドは、非同期のメッセージ キューを介してワーカーと通信します。 

Web キューワーカーは、リソース集中型タスクを含む比較的単純なドメインに適しています。 n 層と同様にわかりやすいアーキテクチャです。 管理対象サービスを使用することで、デプロイと操作が簡単になります。 ただし、複雑なドメインでは依存関係の管理が難しくなることがあります。 フロント エンドおよびワーカーが、管理と更新の無図解しい大きなモノリシック コンポーネントになりやすいためです。 n 層の場合と同じく、これにより頻繁な更新が減少し、革新が制限されます。

### <a name="microservices"></a>マイクロサービス

<img src="./images/microservices-sketch.svg" style="float:left; margin-top:6px;"/>

アプリケーションのドメインがさらに複雑な場合には、**[マイクロサービス][microservices]** アーキテクチャへの移行を検討してください。 マイクロサービス アプリケーションは、小規模の独立したサービス多数で構成されます。 各サービスが 1 つのビジネス機能を実装します。 サービス同士はゆるやかに結び付いており、API コントラクトを介してやりとりします。

各サービスは小規模の専門開発チームによって構築できます。 個々 のサービスをデプロイする際にチーム間での調整があまり必要ないため、頻繁な更新が促進されます。 マイクロサービス アーキテクチャは、n 層または Web キューワーカーに比べて構築や管理が複雑です。 成熟した開発技術および DevOps カルチャが必要です。 ただし、適切に行えば、このスタイルによってリリースにかかる時間が短縮され、革新が進み、回復力のあるアーキテクチャが実現します。 

### <a name="cqrs"></a>CQRS

<img src="./images/cqrs-sketch.svg" style="float:left; margin-top:6px;"/>

**[CQRS](./cqrs.md)** (コマンド クエリ責務分離) スタイルでは、読み取り操作と書き込み操作を別のモデルに分離されます。 これにより、データを更新するシステムの部分がデータを読み取る部分から切り離されます。 さらに、書き込み用データベースから物理的に分離された具体化されたビューに対して読み取りを実行できます。 これで、読み取りと書き込みのワークロードを個別に拡張でき、具体化されたビューをクエリに合わせて最適化できます。

CQRS は、大規模アーキテクチャのサブシステムへの適用が適しています。 通常、アプリケーション全体への適用はお勧めしていません。複雑になるだけでメリットがないためです。 多くのユーザーが同じデータにアクセスするコラボレーション ドメインでの使用を検討してください。

### <a name="event-driven-architecture"></a>イベントドリブン アーキテクチャ 

<img src="./images/event-driven-sketch.svg" style="float:left; margin-top:6px;"/>

**[イベントドリブン アーキテクチャ](./event-driven.md)**は、プロデューサーがイベントをパブリッシュし、コンシューマーがサブスクライブするパブリッシュ - サブスクライブ (pub-sub) モデルを使用します。 プロデューサーはコンシューマーとは独立しており、各コンシューマーは互いに独立しています。 

イベントドリブン アーキテクチャは、ごく短い待機時間で大容量のデータを取り込んで処理するアプリケーション (IoT ソリューションなど) について検討してください。 このスタイルは、さまざまサブシステムが同じイベント データに対してさまざまな種類の処理を実行する必要がある場合にも役立ちます。

<br />
### <a name="big-data-big-compute"></a>ビッグ データ、ビッグ コンピューティング

**[ビッグ データ](./big-data.md)**および**[ビッグ コンピューティング](./big-compute.md)**は、特定のプロファイルと一致するワークロードに特化したアーキテクチャ スタイルです。 ビッグ データでは、きわめて大きなデータセットをチャンクに分割して、分析やレポート作成のためにセット全体に並列処理を実行します。 ビッグ コンピューティングはハイ パフォーマンス コンピューティング (HPC) とも呼ばれ、多数 (数千単位) のコアに対して並列コンピューティングを行います。 ドメインには、シミュレーション、モデリング、および 3-D レンダリングが含まれます。

## <a name="architecture-styles-as-constraints"></a>制約としてのアーキテクチャ スタイル

アーキテクチャ スタイルによって設計 (含めることができる要素のセットや、それらの要素間で許可される関係など) が制約されます。 制約によって、選択肢の範囲が狭められ、アーキテクチャの "シェイプ" が決まります。 アーキテクチャがあるスタイルの制約に準拠すると、特定の望ましいプロパティが出現します。 

たとえば、マイクロサービスの制約は次のとおりです。 

- 1 つのサービスが 1 つの役割を表します。 
- すべてのサービスは互いに独立しています。 
- データはそれを所有するサービス専用です。 サービスは、データを共有しません。

このような制約に準拠することで出現するシステムでは、サービスを個別にデプロイでき、障害が分離され、頻繁な更新が可能になります。また、新しいテクノロジをアプリケーションに容易に導入できます。

アーキテクチャ スタイルを選択する前に、そのスタイルの基本原則と制約を理解してください。 そうしないと、表面的なレベルでスタイルに準拠した設計ができあがり、そのスタイルの潜在力すべてを引き出すことはできません。 実際的に対処することも重要です。 アーキテクチャの正しい形にこだわるよりも、場合によっては制約を緩和する方が適しています。


次の表には、各スタイルでの依存関係の管理方法と各スタイルに最適なドメインの種類をまとめています。

| アーキテクチャ スタイル |  依存関係の管理 | ドメインの種類 |
|--------------------|------------------------|-------------|
| n 層 | サブネットごとに分割された水平の層 | 従来のビジネス ドメイン。 更新の頻度は低いです。 |
| Web キューワーカー | 非同期メッセージングによって分離されているフロント ジョブとバックエンド ジョブ。 | ある程度のリソース集中型タスクを含む比較的単純なドメイン。 |
| マイクロサービス | API を介して互いに呼び出しを行う、垂直方向 (機能別) に分解されたサービス。 | 複雑なドメイン。 頻繁に更新されます。 |
| CQRS | 読み取り/書き込みの分離。 スキーマとスケールは個別に最適化されます。 | 多数のユーザーが同じデータにアクセスするコラボレーション ドメイン。 |
| イベントドリブン アーキテクチャ。 | プロデューサー/コンシューマー。 サブシステムごとに独立したビュー。 | IoT およびリアルタイムのシステム |
| ビッグ データ | 大きなデータセットを小さなチャンクに分割します。 ローカル データセットを並列処理します。 | バッチおよびリアルタイムのデータ分析。 ML を使用した予測分析。 |
| ビッグ コンピューティング| 数千のコアへのデータの割り当て。 | シミュレーションなどコンピューティング集中型ドメイン。 |


## <a name="consider-challenges-and-benefits"></a>課題と利点の検討

制約によって課題も生じます。これらのうちどのスタイルを採用する場合でもトレードオフを理解することが重要です。 "*このサブドメインと制限付きコンテキストについて*"、アーキテクチャ スタイルの利点が課題を上回るようにしてください。 

アーキテクチャ スタイルを選択するときに検討する必要がある課題の一部を次に示します。

- **複雑さ**。 アーキテクチャの複雑さがドメインに対して正当かどうか。 逆に、ドメインに対してスタイルが単純すぎないか。 この場合は、"[泥だんご][ball-of-mud]" ができあがるリスクがあります。アーキテクチャが依存関係を適切に管理できないためです。

- **非同期メッセージングと最終的な一貫性**。 非同期メッセージングを使用すると、サービスを切り離すことができ、信頼性 (メッセージは再試行可能) とスケーラビリティを向上できます。 ただし、これにも、always-once セマンティクスや最終的な一貫性などの課題があります。

- **サービス間の通信**。 アプリケーションを個々のサービスに分割するため、サービス間の通信によって、許容できない待ち時間が発生したり、ネットワークの混雑が生じたりするリスクがあります (たとえばマイクロサービス アーキテクチャ)。 

- **管理のしやすさ**。 アプリケーションの管理、監視、更新のデプロイなどがどれくらい困難か。


[ball-of-mud]: https://en.wikipedia.org/wiki/Big_ball_of_mud
[microservices]: ./microservices.md
[n-tier]: ./n-tier.md