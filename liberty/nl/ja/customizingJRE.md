---

copyright:
  years: 2015, 2017
lastupdated: "2017-10-26"

---

{:new_window: target="_blank"}
{:codeblock: .codeblock}

# JRE のカスタマイズ
{: #customizing_jre}

アプリケーションは、Liberty ビルドパックによって提供および構成される Java ランタイム環境 (JRE) で実行されます。Liberty ビルドパックにより、JRE のバージョンまたはタイプの構成、JVM オプションのカスタマイズ、JRE 機能のオーバーレイも可能になります。

## {{site.data.keyword.IBM_notm}} JRE

デフォルトでは、アプリケーションは軽量版の {{site.data.keyword.IBM}} JRE で実行するように構成されます。この軽量 JRE は、
中核の重要な機能のみを提供するようにしたもので、ディスクおよびメモリーにおける占有スペースが大幅に削減されます。 軽量 JRE の内容について詳しくは、[Small Footprint JRE](http://www.ibm.com/support/knowledgecenter/SSYKE2_8.0.0/com.ibm.java.lnx.80.doc/user/small_jre.html) を参照してください。

 デフォルトでは {{site.data.keyword.IBM_notm}} JRE バージョン 8 が使用されます。別のバージョンの {{site.data.keyword.IBM_notm}} JRE を指定するには、JBP_CONFIG_IBMJDK 環境変数を使用します。例えば、最新の {{site.data.keyword.IBM_notm}} JRE 7.1 を使用するには、以下の環境変数を設定します。
```
    $ cf set-env myapp JBP_CONFIG_IBMJDK "version: 1.7.+"
```
{: codeblock}

version プロパティーをバージョン範囲に設定することができます。 サポートされているバージョン範囲は 1.7.+ と 1.8.+ の 2 つです。 最高の結果を得るには Java 8 を使用してください。

## OpenJDK
{: #openjdk}

オプションで、JRE として OpenJDK を使用して実行するようにアプリケーションを構成できます。 アプリケーションを OpenJDK を使用して実行できるようにするには、JVM 環境変数を「openjdk」に設定します。 例えば、
cf コマンド・ライン・ツールを使用して、次のコマンドを実行します。
```
    $ cf set-env myapp JVM 'openjdk'
```
{: codeblock}

デフォルトでは、使用可能であれば OpenJDK バージョン 8 が使用されます。 別のバージョンの OpenJDK を指定するには、JBP_CONFIG_OPENJDK  環境変数を使用します。 例えば、最新の OpenJDK 7 を使用するには、以下の環境変数を設定します。
```
    $ cf set-env myapp JBP_CONFIG_OPENJDK "version: 1.7.+"
```
{: codeblock}

version プロパティーは、バージョン範囲 (例えば 1.7.+) に設定するか、[使用可能な OpenJDK バージョンのリスト](https://download.run.pivotal.io/openjdk/lucid/x86_64/index.yml)に示されている、任意の特定のバージョンに設定することができます。 最高の結果を得るには Java 8 を使用してください。

## Oracle JRE
{: #oracle_jre}

Oracle JRE の使用については、[『Oracle JRE の使用』](oracle_jre.html)を参照してください。

## JRE オプションの構成
{: #configuring_jre}

### JVM デフォルト構成
{: #jvm_default_config}

Liberty ビルドパックにより、以下を考慮して、デフォルト JVM オプションが構成されます。

* アプリケーションのメモリー制限。  適用される JVM ヒープ設定は、以下に基づいて計算されます。
  * アプリケーションのメモリー制限 ([『メモリー制限および Liberty ビルドパック』](memoryLimits.html#memory_limits)を参照)
  * JRE のタイプ (JVM のヒープ関連オプションは JRE でサポートされるオプションによって異なるため)

* [{{site.data.keyword.Bluemix_notm}} でサポートされている Liberty フィーチャー](libertyFeatures.html#libertyfeatures)。
  * {{site.data.keyword.Bluemix_notm}} では 2 フェーズ・コミット・グローバル・データベース・トランザクションはサポートされないため、-Dcom.ibm.tx.jta.disable2PC=true を設定して無効にされます。

* {{site.data.keyword.Bluemix_notm}} 環境。

    JVM オプションは、{{site.data.keyword.Bluemix_notm}} 環境で最適化されるように、さらに、メモリー関連のエラー状態の診断に役立つように構成されます。
  * JVM ダンプ・オプションを無効にし、アプリケーションのメモリーが枯渇したときにプロセスを kill することで、アプリケーションの迅速な障害復旧が構成されます。
  * 仮想化チューニング ({{site.data.keyword.IBM_notm}} JRE のみ)。
  * 障害発生時のアプリケーションの使用可能メモリー・リソースに関する情報を Loggregator にルーティングします。
  * JVM メモリー・ダンプを有効にするようにアプリケーションが構成されている場合、Java プロセスの kill は無効にされ、JVM メモリー・ダンプは共通のアプリケーション「dumps」ディレクトリーにルーティングされます。 このダンプは、{{site.data.keyword.Bluemix_notm}} ダッシュボードまたは Cloud Foundry CLI で表示できます。

以下に、512 M のメモリー制限を指定してデプロイされたアプリケーションに対してビルドパックが生成したデフォルト JVM 構成例を示します。

```
    -Xtune:virtualized
    -Xmx384M
    -Xdump:none
    -Xdump:heap:defaults:file=../../../../../dumps/heapdump.%Y%m%d.%H%M%S.%pid.%seq.phd
    -Xdump:java:defaults:file=../../../../../dumps/javacore.%Y%m%d.%H%M%S.%pid.%seq.txt
    -Xdump:snap:defaults:file=../../../../../dumps/Snap.%Y%m%d.%H%M%S.%pid.%seq.trc
    -Xdump:tool:events=systhrow,filter=java/lang/OutOfMemoryError,request=serial+exclusive,exec=../../../../.buildpack-diagnostics/killjava.sh
    -Dcom.ibm.tx.jta.disable2PC=true
```
{: codeblock}

### JVM 構成のカスタマイズ
{: #customizing_jvm}

アプリケーションでは、アプリケーションに対して構成された JRE によって定義された仕様を使用して、JVM オプションをカスタマイズできます。 オプションは JRE によって異なるため、オプションを指定する方法およびその使用法のガイドラインについては、JRE の資料を直接参照してください。

<table>
<tr>
<th align="left">JRE</th>
<th align="left">コマンド・ライン・オプションの形式</th>
<th align="left">参照</th>
</tr>

<tr>
<td> {{site.data.keyword.IBM_notm}}JRE</td>
<td>ランタイム・オプション (接頭部: -X)、Java システム・プロパティー (接頭部: -D) が含まれます。-XX を無計画に使用することはお勧めできません (このオプションは変更されることがあります)
</td>
<td>[バージョン 8 コマンド・ライン・オプション](http://www-01.ibm.com/support/knowledgecenter/SSYKE2_8.0.0/com.ibm.java.lnx.80.doc/diag/appendixes/cmdline/cmdline.html)、 [バージョン 7 コマンド・ライン・オプション](http://www-01.ibm.com/support/knowledgecenter/SSYKE2_7.0.0/com.ibm.java.lnx.70.doc/diag/appendixes/cmdline/cmdline.html)
</td>
</tr>

<tr>
<td> OpenJDK </td>
<td>HotSpot ランタイムに基づきます。このランタイムの表記には、非標準の -X、開発者オプションの -XX、およびオプションを有効または無効にするブール・フラグがあります </td>
<td>[HotSpot ランタイムの概要  ![外部リンク・アイコン](../../icons/launch-glyph.svg "外部リンク・アイコン")](http://openjdk.java.net/groups/hotspot//docs/RuntimeOverview.html) </td>
</tr>
</table>

JVM オプションのカスタマイズを必要とするアプリケーションは、{{site.data.keyword.Bluemix_notm}} で環境変数 IBM_JAVA_OPTIONS、JAVA_OPTS、または JVM_ARGS のいずれかの値としてオプションを設定できます。アプリケーションの環境変数の設定方法については、セクション『環境変数』を参照してください。 環境変数を設定する代わりに、パッケージされたサーバーまたはサーバーのディレクトリーに、コマンド・ライン・オプションが入っている jvm.options ファイルを含めることもできます。

JVM オプションが JRE に適用される際に、Liberty ビルドパックのデフォルト・オプションが先に適用され、続けてカスタマイズしたオプションが適用されます。 カスタマイズしたオプションは、以下の表でリストしている特定の順序で追加されます。 適用された Java オプションの順序により、優先されるオプションが決定されます。 最後に適用されたオプションが、それより前に適用されたオプションより優先されます。

注: 一部のオプションは、エージェントによってトリガーされない限り、有効にならないことがあります。

<table>
<tr>
<th align="left">適用の順序</th>
<th align="left">アプリケーションの JVM オプション設定</th>
<th align="left">説明</th>
<th align="left">サポートされるアプリケーション・タイプ</th>
<th align="left">デプロイ済みアプリケーションの JVM オプションを更新する方法</th>
<th align="left">OpenJDK JRE への適用可能性</th>
</tr>

<tr>
<td>1</td>
<td>IBM_JAVA_OPTIONS</td>
<td>{{site.data.keyword.IBM_notm}} JRE によってサポートされる環境変数</td>
<td>すべて</td>
<td>アプリケーションの再始動または再ステージング</td>
<td>いいえ</td>
</tr>

<tr>
<td>2</td>
<td>JAVA_OPTS</td>
<td>Liberty ビルドパック Java オプション・フレームワークによる環境変数</td>
<td>すべて</td>
<td>アプリケーションの再ステージング</td>
<td>はい</td>
</tr>

<tr>
<td>3</td>
<td>jvm.options</td>
<td>Liberty ランタイムのパッケージされたサーバーまたはサーバーのディレクトリーによってサポートされる JVM 構成ファイル</td>
<td>サーバー・パッケージ</td>
<td>アプリケーションの再ステージング</td>
<td>はい</td>
</tr>

<tr>
<td>4</td>
<td>JVM_ARGS</td>
<td>Liberty ランタイムによってサポートされる環境変数</td>
<td>すべて</td>
<td>アプリケーションの再始動または再ステージング</td>
<td>はい</td>
</tr>
</table>

### 実行中のアプリケーションに適用されている JVM オプションの判別
{: #determining_applied_jvm_options}

JVM_ARGS 環境変数を使用して指定されたアプリケーション定義オプションを除いて、結果のオプションは、ランタイム環境で、コマンド・ライン・オプションとして (スタンドアロン Java アプリケーション)、または `jvm.options` ファイル内に (非スタンドアロン Java アプリケーション) 保持されます。 アプリケーションに適用されている JVM オプションは、{{site.data.keyword.Bluemix_notm}} コンソールまたは Cloud Foundry CLI で表示できます。

スタンドアロン Java アプリケーションの JVM オプションは、
コマンド・ライン・オプションとして保持されます。 `staging_info.yml` ファイルから表示できます。

DEA ノードで実行されているアプリケーションの `staging_info.yml` ファイルを表示するには、以下を実行します。

```
    $ cf files myapp staging_info.yml
```
{: codeblock}

Diego セルで実行されているアプリケーションの `staging_info.yml` ファイルを表示するには、以下を実行します。

```
    $ cf ssh myapp -c "cat staging_info.yml"
```
{: codeblock}

WAR、
EAR、サーバー・ディレクトリー、およびパッケージされたサーバーのデプロイメントの場合、JVM オプションは `jvm.options` ファイルで保持されます。 `jvm.options` ファイルは、`app/wlp/usr/servers/<serverName>/` ディレクトリーにあります。 パッケージされたサーバーが別のサーバー名でデプロイされた場合を除いて、```<serverName>``` は `defaultServer` に設定されるのが一般的です。 例えば、次のとおりです。

DEA ノードで実行されているアプリケーションの `jvm.options` ファイルを表示するには、以下を実行します。

```
    $ cf files myapp app/wlp/usr/servers/defaultServer/jvm.options
```
{: codeblock}

Diego セルで実行されているアプリケーションの `jvm.options` ファイルを表示するには、以下を実行します。

```
    $ cf ssh myapp -c "cat app/wlp/usr/servers/defaultServer/jvm.options"
```
{: codeblock}


#### 使用例
{: #example_usage}

{{site.data.keyword.IBM_notm}} JRE 冗長ガーベッジ・コレクション・ロギングを有効にするためにカスタマイズした JVM オプションを指定してアプリケーションをデプロイする場合:
* アプリケーションの `manifest.yml` ファイルに含まれる JVM オプション:

```
    env:
      JAVA_OPTS: "-verbose:gc -Xverbosegclog:./verbosegc.log,10,1000"
```
{: codeblock}

* DEA ノードで実行されているアプリケーションの、JVM が生成した冗長ガーベッジ・コレクション・ログ・ファイルを表示するには、以下を実行します。

```
    $ cf files myapp app/wlp/usr/servers/defaultServer/verbosegc.log.001
```
{: codeblock}

* Diego セルで実行されているアプリケーションの、JVM が生成した冗長ガーベッジ・コレクション・ログ・ファイルを表示するには、以下を実行します。

```
    $ cf ssh myapp -c "cat app/wlp/usr/servers/defaultServer/verbosegc.log.001"
```
{: codeblock}

* OutOfMemory 条件に基づいて heap、snap、および javacore をトリガーするように、デプロイ済みアプリケーションの {{site.data.keyword.IBM_notm}} JRE オプションを更新するには、 以下のように JVM オプションを指定してアプリケーションの環境変数を設定し、アプリケーションを再始動します。

```
    $ cf set-env myapp JVM_ARGS '-Xdump:heap+java+snap:events=systhrow,filter=java/lang/OutOfMemoryError'
    $ cf restart myapp
```
{: codeblock}

 生成されたダンプ・ファイルの表示およびダウンロードについて詳しくは、[ロギングおよびトレース](loggingAndTracing.html#download_dumps)の説明を参照してください。

### JRE のオーバーレイ
{: #overlaying_jre}

機能を公開するためにファイルを JRE にバンドルする必要があるケースがあります。 アプリケーション開発者は、カスタマイズ用に JRE ファイルを提供することができます。

オーバーレイされるファイルは、アーカイブのルートにある resources フォルダー内に、アプリケーションの WAR、EAR、または JAR と一緒にパッケージすることができます。 サーバー (圧縮ファイルまたはサーバー・ディレクトリー) の場合、ファイルは、サーバー・ディレクトリーの resources フォルダー内に server.xml ファイルと一緒にパッケージできます。

* WAR ファイル
  * WEB-INF
  * resources
    * other files
    * .java-overlay


* EAR ファイル
  * META-INF
  * resources
    * other files
    * .java-overlay


* JAR ファイル
  * META-INF
  * resources
    * other files
    * .java-overlay


* server_name DIR
  * apps
  * dropin
  * server.xml
  * resources
    * other files
    * .java-overlay

.java-overlay ディレクトリーには、先頭が .java/jre のオーバーレイ対象 Java JRE と同じファイル階層で、特定のファイルが含まれます。

例えば、AES 256 ビット暗号化を使用する場合、以下の Java ポリシー・ファイルをオーバーレイする必要があります。

```
    .java\jre\lib\security\US_export_policy.jar
    .java\jre\lib\security\local_policy.jar
```
{: codeblock}

適切な非制限ポリシー・ファイルをダウンロードし、それらを次のファイルとしてアプリケーションに追加します。

```
    resources\.java-overlay\.java\jre\lib\security\US_export_policy.jar
    resources\.java-overlay\.java\jre\lib\security\local_policy.jar
```
{: codeblock}

アプリケーションをプッシュすると、これらの jar が Java ランタイムのデフォルトのポリシー jar をオーバーレイします。 このプロセスにより、AES 256 ビット暗号化が有効になります。

# 関連リンク
{: #rellinks notoc}
## 一般
{: #general notoc}
* [Liberty ランタイム](index.html)
* [Liberty プロファイル概要](http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.wlp.nd.doc/ae/cwlp_about.html)
