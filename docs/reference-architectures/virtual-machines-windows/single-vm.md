---
title: "Azure での Windows VM の実行"
description: "スケーラビリティ、回復性、管理容易性、およびセキュリティに注意しながら、Azure で Windows VM を実行する方法。"
author: telmosampaio
ms.date: 12/12/2017
pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: ffc8ddcbdd5422f1e38922fc6735ab1579289c7b
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2018
---
# <a name="run-a-windows-vm-on-azure"></a>Azure での Windows VM の実行

この参照アーキテクチャは、Azure で Windows 仮想マシン (VM) を実行するための一連の実証済みの方法を示しています。 ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。 このアーキテクチャは単一の VM インスタンスを実行するために使用でき、N 層アプリケーションなどのさらに複雑なアーキテクチャの基礎となります。 [**以下のソリューションをデプロイします。**](#deploy-the-solution)

![[0]][0]

*このアーキテクチャ ダイアグラムを含む [Visio ファイル][visio-download]をダウンロードします。*

## <a name="architecture"></a>アーキテクチャ

Azure VM のプロビジョニングには、コンピューティング、ネットワーク、およびストレージ リソースなどの追加コンポーネントが必要です。

* **リソース グループ。** [リソース グループ][resource-manager-overview]は、関連リソースを保持するコンテナーです。 一般には、リソースの有効期間や、そのリソースを誰が管理するかに基づいてソリューション内のリソースをグループ化する必要があります。 単一の VM ワークロードの場合は、すべてのリソースに対して単一のリソース グループを作成することもできます。
* **VM**。 VM は、発行されたイメージの一覧や、Azure BLOB ストレージにアップロードされたカスタム管理されたイメージまたは仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。
* **OS ディスク。** OS ディスクは [Azure Storage][azure-storage] に格納された VHD であるため、ホスト マシンが停止している場合でも維持されます。
* **一時ディスク。** VM は一時ディスク (Windows の `D:` ドライブ) を使用して作成されます。 このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。 Azure Storage には保存され**ない**ため、再起動やその他の VM ライフサイクル イベント中に削除される可能性があります。 ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。
* **データ ディスク。** [データ ディスク][data-disk]は、アプリケーション データ用の永続的な VHD です。 データ ディスクは、OS ディスクと同様に、Azure Storage に格納されます。
* **仮想ネットワーク (VNet) とサブネット。** どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。
* **パブリック IP アドレス。** パブリック IP アドレスは、リモート デスクトップ (RDP) 経由などで VM &mdash; と通信するために必要です。  
* **Azure DNS**。 [Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。  
* **ネットワーク インターフェイス (NIC)**。 割り当てられた NIC により、VM は仮想ネットワークと通信できます。  
* **ネットワーク セキュリティ グループ (NSG)**。 [ネットワーク セキュリティ グループ][nsg]は、ネットワーク リソースへのネットワーク トラフィックを許可または拒否するために使用されます。 NSG は、個々 の NIC またはサブネットに関連付けることができます。 NSG をサブネットに関連付けると、そのサブネット内のすべての VM に NSG ルールが適用されます。
* **[診断]。** 診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。

## <a name="recommendations"></a>推奨事項

このアーキテクチャでは、Azure で Windows VM を実行するためのベースラインの推奨事項が示されます。 ただし、単一障害点ができてしまうため、ミッション クリティカルなワークロードに単一の VM を使用することは推奨されません。 可用性を高めるには、複数の VM を[可用性セット][availability-set]にデプロイします。 詳細については、[Azure での複数の VM の実行][multi-vm]に関するページをご覧ください。 

### <a name="vm-recommendations"></a>VM の推奨事項

Azure は、さまざまな仮想マシン サイズを提供します。 [Premium Storage][premium-storage] は高パフォーマンスと短い待機時間のために推奨され、[特定の VM サイズでサポートされます][premium-storage-supported]。 ハイ パフォーマンス コンピューティングなどの特殊なワークロードがない限り、これらのサイズのいずれかを選択してください。 詳細については、「[仮想マシン サイズ][virtual-machine-sizes]」をご覧ください。

既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。 次に、CPU、メモリ、およびディスクの 1 秒あたりの入出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。 VM 用に複数の NIC が必要な場合は、[VM サイズ][vm-size-tables]ごとに NIC の最大数が定義されていることに注意してください。

Azure リソースをプロビジョニングする場合は、リージョンを指定する必要があります。 一般的に、内部ユーザーや顧客に最も近いリージョンを選択します。 ただし、すべてのリージョンですべての VM サイズを使用できるわけではありません。 詳細については、「[リージョン別のサービス][services-by-region]」をご覧ください。 特定のリージョンで使用できる VM サイズの一覧を表示するには、Azure コマンド ライン インターフェイス (CLI) から次のコマンドを実行します。

```
az vm list-sizes --location <location>
```

発行された VM イメージの選択については、「[Find Windows VM images (Windows VM イメージを検索する)][select-vm-image]」をご覧ください。

基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。 VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。 詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。  

### <a name="disk-and-storage-recommendations"></a>ディスクとストレージの推奨事項

最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。 コストは、プロビジョニングされたディスクの容量に基づきます。 また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。 

[管理ディスク](/azure/storage/storage-managed-disks-overview)を使用することもお勧めします。 管理ディスクでは、ストレージ アカウントは必要ありません。 単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。

非管理対象ディスクを使用している場合は、ストレージ アカウントの [ (IOPS) の制限][vm-disk-limits]に達しないようにするために、仮想ハード ディスク (VHD) を保持するための個別の Azure ストレージ アカウントを VM ごとに作成します。

1 つ以上のデータ ディスクを追加します。 作成した VHD は、フォーマットされていません。 その VM にログインしてディスクをフォーマットしてください。 管理ディスクを使用せず、データ ディスクの数が多い場合は、ストレージ アカウントの合計 I/O 制限に注意してください。 詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。

可能であれば、OS ディスクではなく、データ ディスクにアプリケーションをインストールします。 一部のレガシー アプリケーションでは、C: ドライブへのコンポーネントのインストールが必要になることがあります。その場合は、PowerShell を使用して [OS ディスクのサイズを変更][resize-os-disk]できます。

パフォーマンスを最大化するには、診断ログを保持するための個別のストレージ アカウントを作成します。 診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。

### <a name="network-recommendations"></a>ネットワークの推奨事項

パブリック IP アドレスは、動的でも静的でもかまいません。 既定では、動的アドレスになっています。

* 変化しない固定 IP アドレスが必要な場合 &mdash; (たとえば、DNS 'A' レコードを作成するか、またはセーフ リストに IP アドレスを追加する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。
* IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。 これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。 詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」をご覧ください。

すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。 既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。 インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。

RDP を有効にするには、TCP ポート 3389 への着信トラフィックを許可する NSG ルールを追加します。

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。 水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。 詳細については、「[Running multiple VM on Azure for scalability and availability (スケーラビリティと可用性のための Azure での複数の VM の実行)][multi-vm]」をご覧ください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

可用性を高めるには、複数の VM を可用性セットにデプロイします。 これにより、より高度な[サービス レベル アグリーメント][vm-sla] (SLA) も提供されます。

VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。 [VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。

VHD は [Azure Storage][azure-storage] に格納されます。 Azure Storage は、持続性と可用性を確保するためにレプリケートされます。

通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

**リソース グループ** 同じライフサイクルを共有する密接に関連付けられたリソースを同じ[リソース グループ][resource-manager-overview]に配置します。 リソース グループを使用すると、リソースをグループとしてデプロイおよび監視したり、リソース グループ別に課金コストを追跡したりできます。 セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。 特定のリソースの検索やその役割の理解を簡略化するために、意味のあるリソース名を割り当ててください。 詳細については、「[Azure リソースの推奨される名前付け規則][naming-conventions]」をご覧ください。

**VM の停止。** Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。 VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。

Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。 ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "**解除されない**" ため、引き続き課金されます。

**VM の削除。** VM を削除しても VHD は削除されません。 つまり、データを失うことなく安全に VM を削除できます。 ただし、Storage に対して引き続き課金されます。 VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。

誤って削除されないようにするために、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、または個別のリソース (VM など) をロックします。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。 Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。 セキュリティ センターは、Azure サブスクリプションごとに構成されます。 「[Azure Security Center クイック スタート ガイド][security-center-get-started]」の説明に従って、セキュリティ データの収集を有効にします。 データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。

**更新プログラムの管理。** 有効になっている場合、Security Center はセキュリティ更新プログラムや緊急更新プログラムが不足しているかどうかをチェックします。 VM で[グループ ポリシー設定][group-policy]を使用して、システムの自動更新を有効にします。

**マルウェア対策。** 有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。 セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。

**操作。** [ロールベースのアクセス制御 (RBAC)][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御します。 RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。 たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。 一部の役割は、特定の Azure リソースの種類に固有です。 たとえば、仮想マシンの共同作業者ロールは VM を再起動または割り当て解除したり、管理者パスワードをリセットしたり、新しい VM を作成したりできます。 このアーキテクチャで役立つ可能性のあるその他の[組み込みの RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワークの共同作業者][rbac-network]が含まれます。 ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。

> [!NOTE]
> RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。 これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。   

プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。

**データの暗号化。** OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。 

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。 以下がデプロイされます。

  * VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。
  * VM に対する RDP と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。
  * Windows Server 2016 Datacenter Edition の最新バージョンを実行する VM。
  * 2 つのデータ ディスクをフォーマットするサンプル カスタム スクリプト拡張機能と、IIS をデプロイする PowerShell DSC スクリプト。

### <a name="prerequisites"></a>前提条件

参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。

1. [AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。 CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。

3. [Azure の構成要素][azbb] npm パッケージをインストールします。

4. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>azbb を使用したソリューションのデプロイ

単一の VM ワークロードのサンプルをデプロイするには、次の手順に従います。

1. 上の前提条件の手順でダウンロードしたリポジトリの `virtual-machines\single-vm\parameters\windows` フォルダーに移動します。

2. `single-vm-v2.json` ファイルを開き、次に示すような引用符の間にユーザー名と SSH キーを入力した後、ファイルを保存します。

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. 次に示すように、`azbb` を実行して VM のサンプルをデプロイします。

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

この参照アーキテクチャのサンプルのデプロイについて詳しくは、[GitHub リポジトリ][git]を参照してください。

## <a name="next-steps"></a>次の手順

- [Azure の構成要素][azbbv2]について確認します。
- Azure で[複数の VM][multi-vm] をデプロイします。

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[multi-vm]: multi-vm.md
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure での単一 Windows VM アーキテクチャ"
