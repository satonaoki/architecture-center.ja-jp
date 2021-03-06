# <a name="data-lakes"></a>データ レイク

データ レイクは、大量のデータを未加工のネイティブ形式で保持するストレージ リポジトリです。 データ レイク ストアは、テラバイト級およびペタバイト級のデータにスケーリングできるように最適化されています。 通常、データは、構造化データ、半構造化データ、または非構造化データを含む複数の異種ソースから取得されます。 データ レイクの着想は、それらすべてを元の未変換状態で格納するという点にあります。 このアプローチは、データを取り込んだ時点で変換と処理を行う、従来の[データ ウェアハウス](../scenarios/data-warehousing.md)とは異なります。

データ レイクの利点は次のとおりです。

- データは未加工の形式で格納されるため、データが破棄されることはありません。 これは特にビッグ データ環境で有用です。ビッグ データ環境では、データからどのような洞察が得られるか、前もってわからないからです。
- ユーザーは、データを探索したり、独自のクエリを作成したりできます。
- 従来の ETL ツールよりも高速に動作する可能性があります。
- 非構造化および半構造化データを格納できるため、データ ウェアハウスよりも柔軟です。 

完全なデータ レイク ソリューションは、ストレージと処理の両方で構成されます。 データ レイク ストレージは、フォールト トレランス、無限のスケーラビリティ、さまざまな形状とサイズのデータ​​の高スループットの取り込みなどを目的としています。 Data Lake 処理には、これらの目標を念頭に置いて構築された 1 つ以上の処理エンジンが関係し、Data Lake に格納された大規模なデータを処理することができます。

## <a name="when-to-use-a-data-lake"></a>データ レイクが適したシナリオ

データ レイクの一般的な用途には、[データ探索](../scenarios/interactive-data-exploration.md)、データ分析、機械学習などがあります。 

データ レイクは、データ ウェアハウスのデータ ソースとして機能させることもできます。 このアプローチでは、生データをデータ レイクに取り込み、次にクエリ可能な構造化された形式に変換します。 通常、この変換において [ELT](../scenarios/etl.md#extract-load-and-transform-elt) (抽出 - 読み込み - 変換) パイプラインが使用されます。このパイプラインにはデータが取り込まれ、適宜変換されます。 既にリレーショナルになっているソース データは、ETL プロセスを使用して、データ レイクをスキップしてデータ ウェアハウスに直接取り込まれる場合があります。

Data Lake ストアは、イベント ストリーミングや IoT のシナリオでよく使用されます。変換やスキーマ定義を使用せずに、大量のリレーショナル データと非リレーショナル データを永続化できるためです。 短い待機時間で小規模な書き込みを大量に処理するために構築されており、大量のスループットに合わせて最適化されています。

## <a name="challenges"></a>課題

- スキーマまたは記述メタデータがない場合、データの使用やクエリが困難になることがあります。
- データ全体に意味論的な一貫性が欠けている場合、ユーザーにデータ分析の高度なスキルがない限り、データ分析の実行が困難になることがあります。
- データ レイクに取り込まれるデータの質を保証することが困難な場合があります。 
- 適切に管理しないと、アクセス制御およびプライバシーに関する問題が発生することがあります。 どの情報をデータ レイクに取り込むか、どのユーザーがそのデータにアクセスできるか、また、どの目的で使用するのかを検討する必要があります。
- 既にリレーショナルになっているデータを統合する方法としては、データ レイクは最善とは言えない場合があります。
- データ レイクだけで、組織全体に渡る包括的で統合された視野が提供されるわけではありません。 
- データ レイクは、洞察を得るために実際に分析やマイニングが行われることのないデータの廃棄場になってしまう可能性があります。

## <a name="relevant-azure-services"></a>関連 Azure サービス

- [Data Lake Store](/azure/data-lake-store/) は、Hadoop 互換のハイパースケール リポジトリです。
- [Data Lake Analytics](/azure/data-lake-analytics/) は、ビッグ データ分析を簡略化するオンデマンド分析ジョブ サービスです。

