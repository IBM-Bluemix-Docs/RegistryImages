---

copyright:
  years: 2018, 2019
lastupdated: "2019-02-21"

keywords: ibmcloud-secure-perimeter-health, container image, health, Secure Perimeter, scan, public image

subcollection: RegistryImages

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:table: .aria-labeledby="caption"}

# `ibmcloud-secure-perimeter-health` イメージの概説
{: #ibmcloud-secure-perimeter-health}

`ibmcloud-secure-perimeter-health` イメージには、{{site.data.keyword.cloud}} 内の Secure Perimeter の脆弱性をスキャンするためのツールが含まれています。
{:shortdesc}

{{site.data.keyword.IBM_notm}} によって提供されるイメージには、コマンド・ラインを使用してアクセスできます。[IBM のパブリック・イメージ](/docs/services/Registry?topic=registry-public_images#public_images)を参照してください。
{: tip}

## 機能
{: #sph_how-it-works}

Secure Perimeter が適切に機能していることを確認するために、`ibmcloud-secure-perimeter-health` で、ご使用の {{site.data.keyword.cloud_notm}} インフラストラクチャー・アカウントのパブリック・ネットワークまたはプライベート・ネットワークをスキャンし、脆弱性のレポートを作成できます。 **ibmcloud-secure-perimeter-health** イメージは、以下の 2 つの方法で使用できます。

- `ibmcloud-secure-perimeter-health` を Secure Perimeter 内の Kubernetes クラスター上のポッドとして使用して、プライベート・ネットワークの露出をスキャンする。
- `ibmcloud-secure-perimeter-health` をワークステーション上のスタンドアロン Docker コンテナーとして使用して、パブリック・ネットワークの露出をスキャンする。

Secure Perimeter について詳しくは、以下のブログ記事を参照してください。

- [Set up a Secure Perimeter in {{site.data.keyword.cloud_notm}} ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/dwblog/2018/ibm-cloud-vyatta-set-up-secure-perimeter/).
- [Set up an automated Secure Perimeter in {{site.data.keyword.cloud_notm}} ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/dwblog/2018/set-automated-secure-perimeter-ibm-cloud/).

スキャンが終了すると、`ibmcloud-secure-perimeter-health` イメージによって、Secure Perimeter Segment 内から到達可能であったネットワークに関するレポートが生成されます。 各レポートには、ネットワーク・ゲートウェイの名前、VLAN、そのサブネット、および問題のあるホストがある場合はそのようなホストが詳細に示されます。 プライベート・ネットワークの脆弱性をスキャンしたユーザーのレポートの例を以下に示します。

```
#-------- Running Secure Perimeter exposure scan 2018-05-24 12:00:00 --------#

RESULTS:

sp-gateway-af6053a9:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   FAIL
			host = 10.73.84.198, ports = [179, 22]
		10.73.63.8/29:     PASS
		10.73.72.128/26:   PASS
sp-gateway-8a9031ab:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   PASS
		10.73.63.8/29:     PASS
```
{: screen}

## 含まれている内容
{: #sph_whats_included}

`ibmcloud-secure-perimeter-health` イメージに、以下のソフトウェア・パッケージが用意されています。
{:shortdesc}

- Alpine Linux
- Python ランタイム
- SoftLayer Python クライアント
- Nmap ポート・スキャナー

## {{site.data.keyword.containerlong_notm}} を使用して Secure Perimeter 内で Kubernetes クラスターをプロビジョンする
{: #sph_provision_cluster}

1. {{site.data.keyword.cloud_notm}} カタログ内の**「コンテナー」**セクションから Kubernetes クラスターをプロビジョンします。
2. **「作成」**をクリックします。
3. VLAN ドロップダウン・メニューから Secure Perimeter Segment パブリックおよびプライベート VLAN を選択します。
4. 必要に応じて他のすべての詳細を入力します。
5. **「クラスターの作成」**をクリックします。

クラスターがデプロイされた後、クラスターにアクセスする方法については、[{{site.data.keyword.containerlong_notm}}](/docs/containers?topic=containers-container_index#container_index) 資料を参照してください。

## Secure Perimeter 内のプライベート・ネットワークをスキャンする
{: #sph_private_networks}

`ibmcloud-secure-perimeter-health` イメージからコンテナー・ポッドを作成し、ルーチン・スキャンをセットアップします。

**開始する前に**

- 必要な [CLI](/docs/containers?topic=containers-cs_cli_install#cs_cli_install) をインストールします。
- [CLI のターゲット](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)を自分のクラスターに設定します。

1. _health-pod.yaml_ という名前の構成ファイルを作成します。 このファイルにより、高可用性のコンテナー・ポッドのデプロイメントが作成されます。

    ```
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: health-pod
      labels:
        app: health-pod
    spec:
      replicas: 1
      selector:
        app: health-pod
      template:
        spec:
          containers:
          - name: health-pod
            image: registry.bluemix.net/ibm/ibmcloud-secure-perimeter-health:1.0.0
            args:
            - /usr/local/bin/python
            - /run.py
            - --scan
            - private
            - --exclude-vlan-ids
            - <Private Secure Perimeter Segment VLAN ID>
            - --poll-interval
            - 1800
            env:
            - name: SL_USER
              value: <IBM Cloud infrastructure user name>
            - name: SL_APIKEY
              value: <IBM Cloud infrastructure api key>
    ```
    {: codeblock}
    
2. デプロイメントを作成します。

    ```
    kubectl apply -f health-pod.yaml
    ```
    {: pre}

3. ポッドが実行中であることを確認します。

    ```
    kubectl get pods
    ```
    {: pre}

    ```
    NAME                                    READY     STATUS    RESTARTS   AGE
    health-pod-<random-id>                  1/1       Running   0          1hr
    ```
    {: screen}

## Secure Perimeter 外部のパブリック・ネットワークをスキャンする
{: #sph_public_networks}

`ibmcloud-secure-perimeter-health` イメージから Docker コンテナーを作成し、パブリック・ネットワークをスキャンします。

**開始する前に**

- Docker をインストールします。

1. 以下のように、自身のワークステーションから Docker コンテナーを作成します。

    ```
    docker run -it -e SL_USER='$SL_USER' -e SL_APIKEY='$SL_APIKEY' registry.bluemix.net/ibm/ibmcloud-secure-perimeter-health:1.0.0 /usr/local/bin/python run.py --scan public --allowed-public-ports 80 443 9000-9999
    ```
    {: pre}

2. コンテナーによってレポートが生成されたら、[スキャン結果の分析](#sph_scan_results)セクションを参照して、結果を理解します。

## スキャン結果の分析
{: #sph_scan_results}

`ibmcloud-secure-perimeter-health` は、以下のように、Secure Perimeter の機能の正常性に関する定形式レポートを生成します。

```
#-------- Running Secure Perimeter exposure scan 2018-05-24 12:00:00 --------#

RESULTS:

sp-gateway-af6053a9:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   FAIL
			host = 10.73.84.198, ports = [179, 22]
		10.73.63.8/29:     PASS
		10.73.72.128/26:   PASS
sp-gateway-8a9031ab:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   PASS
		10.73.63.8/29:     PASS
```
{: screen}

レポートのフォーマットは、以下のとおりです。

```
<gateway name>:
  <vlan name>:
    <subnet>:    PASS
    <subnet>:    FAIL
      host = <ip address>, ports = [<exposed port>, <exposed port>, ...]
```
{: screen}

`ibmcloud-secure-perimeter-health` は、サブネット内に到達可能なホストがなかった場合にはサブネットを `PASS` と判定し、そうでなければ `FAIL` を返し、到達可能であったホストとアクセス可能であったポートをリストします。

## コンテナー引数リファレンス
{: #sph_reference_container_arg}

|キー|説明|デフォルト
|---|-------------|---|
|scan|露出スキャンのタイプ (「public」または「private」) |なし (両方をスキャン)
|exclude-vlan-ids|スキャンを回避する VLAN ID のリスト|なし
|poll-interval|次のスキャンまでの秒数を設定|0 (実行は 1 回)
|allowed-public-ports|スキャンでのホワイトリストに登録するポートのリスト|80, 443, 9000-9999
{: caption="表 1. コンテナー引数" caption-side="top"}

## 環境変数リファレンス
{: #sph_reference_env_var}

|キー|説明|
|---|-------------|
|SL_USER|IBM Cloud インフラストラクチャー・ユーザー名|
|SL_APIKEY|IBM Cloud インフラストラクチャー API 鍵|
{: caption="表 2. 環境変数" caption-side="top"}
