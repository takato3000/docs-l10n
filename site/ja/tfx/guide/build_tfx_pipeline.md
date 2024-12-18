# TFX パイプラインを構築する

注意: TFX パイプラインの概念図については、[TFX パイプラインを理解する](understanding_tfx_pipelines)を参照してください。

注意: 詳しい内容に掘り下げる前に、まずはパイプラインを構築してみようと思う方は、[テンプレートを使ってパイプラインを構築する](https://www.tensorflow.org/tfx/guide/build_local_pipeline#build_a_pipeline_using_a_template)ことから始めてください。

## `Pipeline` クラスの使用

TFX パイプラインは、[`Pipeline` クラス](https://github.com/tensorflow/tfx/blob/master/tfx/orchestration/pipeline.py){: .external } を使って定義されます。次の例では `Pipeline` クラスの使用方法を実演しています。

<pre class="devsite-click-to-copy prettyprint">pipeline.Pipeline(
    pipeline_name=&lt;var&gt;pipeline-name&lt;/var&gt;,
    pipeline_root=&lt;var&gt;pipeline-root&lt;/var&gt;,
    components=&lt;var&gt;components&lt;/var&gt;,
    enable_cache=&lt;var&gt;enable-cache&lt;/var&gt;,
    metadata_connection_config=&lt;var&gt;metadata-connection-config&lt;/var&gt;,
)
</pre>

上記のコードから、次の項目を置き換えてください。

- <var>pipeline-name</var>: このパイプラインの名前です。パイプライン名は一意である必要があります。

    TFX は、パイプライン名を使用し、ML メタデータにコンポーネント入力アーティファクトをクエリします。パイプライン名を再利用すると、予期しない動作が発生する可能性があります。

- <var>pipeline-root</var>: このパイプラインの出力のルートパスです。ルートパスは、オーケストレータが読み取りと書き込みのアクセス権をもつディレクトリのフルパスでなければなりません。TFX はランタイム時にこのパイプラインルートを使用して、コンポーネントアーティファクトの出力パスを生成します。ローカル上のディレクトリ、または Google Cloud Storage や HDFS などのサポートされた分散ファイルシステム上のディレクトリを使用できます。

- <var>components</var>: このパイプラインのワークフローを構成するコンポーネントインスタンスのリストです。

- <var>enable-cache</var>:（オプション）パイプライン実行を高速化するために、このパイプラインでキャッシュを使用するかどうかを指定するブール型の値です。

- <var>metadata-connection-config</var>:（オプション）ML メタデータの接続構成です。

## コンポーネント実行グラフを定義する

コンポーネントインスタンスは、アーティファクトを出力として生成し、通常は上流のコンポーネントインスタンスで入力として生成されたアーティファクトに依存します。コンポーネントインスタンスの実行シーケンスは、アーティファクト依存関係の有向非巡回グラフ（DAG）を作成することで決定されます。

たとえば、`ExampleGen` 標準コンポーネントは CSV ファイルからデータを取り込み、シリアル化されたサンプルレコードを出力します。`StatisticsGen` 標準コンポーネントはそのサンプルレコードを入力として受け取り、データセットの統計を生成します。この例では、`SchemaGen` が `ExampleGen` の出力に依存しているため、`StatisticsGen` のインスタンスは `ExampleGen` の後に続く必要があります。

### タスクベースの依存関係

注意: 通常、タスクベースの依存関係の使用は推奨されません。アーティファクト依存関係で実行グラフを定義すると、TFX のアーティファクト系統の自動トラッキングとキャシング機能を利用できます。

また、コンポーネントの[`add_upstream_node` と `add_downstream_node`](https://github.com/tensorflow/tfx/blob/master/tfx/components/base/base_node.py){: .external } メソッドを使って、タスクベースの依存関係を定義することもできます。現在のコンポーネントを指定されたコンポーネントの後に実行する場合は `add_upstream_node`、現在のコンポーネントを指定されたコンポーネントの前に実行する場合は `add_upstream_node` を使用します。

## パイプラインテンプレート

パイプラインを迅速にセットアップし、すべての要素がどのように組み合わされているかを簡単に確認するには、テンプレートを使用します。テンプレートの使用については、[ローカルでの TFX パイプラインの構築](build_local_pipeline)で説明されています。

## キャッシング

TFX パイプラインをキャッシングすると、前回のパイプライン実行で同一の入力セットを使って実行されたコンポーネントをパイプラインから省略することができます。キャッシングが有効になっている場合、パイプラインは各コンポーネントのシグネチャ、コンポーネント、および入力セットをこのパイプラインの前回のコンポーネント実行に一致させようとします。一致する場合、パイプラインは前回の実行で得たコンポーネント出力を使用し、一致しない場合は、コンポーネントを実行します。

パイプラインに非確定的コンポーネントを使用している場合は、キャッシングを使用しないでください。たとえば、パイプラインの乱数を作成するコンポーネントを作成した場合、キャッシングが有効になっていると、このコンポーネントは一度しか実行されません。この場合、後続の実行では、乱数を生成する代わりに、初回の実行で得た乱数が使用されてしまいます。
