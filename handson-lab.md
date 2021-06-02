**目次**

* 演習 1: Azure にリソースを展開する
    * タスク 1: Azure Database for PostgreSQL を展開する
    * タスク 2: Azure 仮想マシンを作成する
    * タスク 3: Azure Database for PostgreSQL へ接続する
    * タスク 4: リソース マネージャー テンプレートを使用して複数のリソースを一括で展開する

* 演習 2: ネットワーク
    * タスク 2: Azure Database for PostgreSQL をプライベート エンドポイントで保護する

* 演習 3: 可用性とスケーラビリティ
    * タスク 1: Azure Database for PostgreSQL をリージョン冗長構成変更にする
    * タスク 2: Web サーバーを Azure フロント ドアでリージョン冗長構成に変更する


**概要**
---
Contoso 社は、既存のオンプレミスのアプリケーションを Microsoft Azure に移行することを検討しています。この際、アプリケーションの変更を最小限に留めながら、できるだけマネージド サービスを活用することを検討してます。

今回、Azure へのマイグレーションに伴い、OS のバージョンや RDBMS のバージョンは最新化することを検討しています。


**ソリューション アーキテクチャ**
---
以下の図は、このラボで構築するアプリケーション アーキテクチャを示しています。Web サーバー、PostgreSQL、だけから始めて、完全に冗長な高可用性環境を東日本リージョンに構築します。次に、この環境を西日本リージョンのセカンダリ サイトに拡張し、Web 層とデータベース層の両方に可用性ソリューションを追加します。

![SolutionOverview](https://raw.githubusercontent.com/tamatsuxyz/azurews/main/images/SolutionOverview.png)
---

## 演習 1: Azure にリソースを展開する
---

Contoso 社の業務アプリケーションは、2階層アーキテクチャであり、RDBMS 製品として PostgreSQL を使用しています。Web アプリケーションは今後 PaaS サービスを活用することを検討していますが、現時点では、既存のパッケージ製品を使用を継続することを検討しています。パッケージ製品のソフトウェア要件で、Web サーバーは Windows OS の仮想マシンを展開する必要があります。


## タスク 1: Azure Database for PostgreSQL を展開する
1. Web ブラウザで Azure ポータルを開きます。(https://portal.azure.com)

   認証画面が表示されたら、サブスクリプションの資格情報を入力して、ログインします。
2. [**＋リソースの作成**] をクリックします。
3. カテゴリから **データベース** を選択し、[**Azure Database for PostgreSQL**] をクリックします。
4. **サービスをどのように使用する予定ですか？** 画面で、**単一サーバー** の [**作成**] ボタンをクリックします。
5. **基本** タブの項目では、以下の情報を入力します。入力後、[**確認および作成**] ボタンをクリックします。
    * リソース グループ: **Prefix**-azurews (新規作成)

        ※ **Prefix** 部分は、任意の文字（ご自身のイニシャルなど）で置き換えてください

    * サーバー名: **Prefix**-jpepsql
    * 地域: 東日本
    * コンピューティングとストレージ: 汎用目的

        ※ **サーバーの構成** をクリックして、以下を指定します。
        * vCore: 2
        * ストレージ 5 GB
        * ストレージの自動拡張: いいえ
    * 管理者ユーザー名: azlabadmin
    * パスワード/パスワードの確認: (**任意のパスワードを入力**)
6. [**作成**] ボタンをクリックします。
7. **デプロイが完了しました** のメッセージが出力されたら [**リソースに移動**] ボタンをクリックします。


## タスク 2: Azure 仮想マシンを作成する
このタスクでは、Web サーバー用の Azure 仮想マシンをデプロイします。また、IIS をインストールした上で外部からアクセスできるようにします。

1. [**＋リソースの作成**] をクリックします。
2. [**Windows Server 2019 Datacenter**] をクリックします。
3. **基本** タブの項目では、以下の情報を入力します。入力後、[**次: ディスク**] ボタンをクリックします。
    * リソース グループ: **Prefix**-azurews (既存）

        ※ タスク1 で作成したリソース グループ名を選択します。

    * 仮想マシン名: WebVM01
    * 地域: 東日本
    * 可用性オプション: インフラストラクチャ冗長は必要ありません
    * サイズ: Standard_E2s_v3
    * ユーザー名: azlabadmin
    * パスワード/パスワードの確認: (**任意のパスワードを入力**)
4. **ディスク** タブの項目では、以下の情報を入力します。入力後、[**次: ネットワーク**] ボタンをクリックします。
    * OS のディスクの種類: Standard HDD
5. **ネットワーク** タブの項目では、以下の情報を入力します。入力後、[**確認および作成**] ボタンをクリックします。
    * 仮想ネットワーク: jpevnet（新規作成）

        ※仮想ネットワークを新規作成する際に、以下の情報を入力します。
        * アドレス範囲: 10.0.0.0/**16**
        * サブネット名: web
        * アドレス空間: 10.0.0.0/**24**
6. **検証に成功しました** のメッセージが出力されたら、[**作成**] ボタンをクリックします。
7. **デプロイが完了しました** のメッセージが出力されたら、[**リソースに移動**] ボタンをクリックします。
8. Azure portal の上部のナビゲーション バーで [Cloud Shell] をクリックして、[PowerShell] を選択します。

    ※ ストレージがマウントされていません のメッセージが表示された場合は、[**ストレージの作成**] をクリックします。
9. 次のコマンドを実行して、IIS を仮想マシンにインストールします。
 
    ※ 以下のコマンドの ResourceGroupName パラメーターをご自身の値に変更してから実行します。
```azurepowershell
 Set-AzVMExtension `
  -ResourceGroupName xx-azurews `
  -ExtensionName IIS `
  -VMName WebVM01 `
  -Publisher Microsoft.Compute `
  -ExtensionType CustomScriptExtension `
  -TypeHandlerVersion 1.4 `
  -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
  -Location JapanEast
```
10. WebVM01 を開き、[設定] - [ネットワーク] をクリックします。
11. **受信ポートの規則を追加する** ボタンをクリックします。
12. 以下の情報を入力し、[追加] ボタンをクリックします。
    * サービス: HTTP
    * アクション: 許可
    * 優先度: 100
    * 名前: AllowHttpInbound
10. **NIC パブリック IP** に表示されているパブリック IP アドレスを Web ブラウザで開き、WebVM01 の文字のみが書かれた Web ページが表示されることを確認します。

※このパブリック IP は、後述の手順でも使用しますのでメモ張などに保存しておきます。

## タスク 3: Azure Database for PostgreSQL へ接続する
このタスクでは、Web サーバーにログインして、Azure Database for PostgreSQL へ接続確認を行います。

1. Azure Database for PostgreSQL の画面を開き、[接続のセキュリティ] をクリックし、WebVM01 の IP アドレスを許可します。
   ファイアウォール規則に以下の値を入力したら、[保存] をクリックします。
    * ファイアウォール規則: WebVM01
    * 開始 IP/終了 IP: (**WebVM01 のパブリック IP**)
2. WebVM01 の画面で[**接続**] をクリックします。
3. [**RDP ファイルのダウンロード**] ボタンをクリックします。
4. ダウンロードした RDP ファイルを開きます。
5. メッセージが表示されたら、[**接続**] をクリックし、仮想マシンの資格情報を入力してログインできることを確認します。 
6. 以下の URL より PostgreSQL クライアントをダウンロードしてインストールします。

　　※ Server Manager で IE のセキュリティ強化機能を事前に無効化が必要です。
https://www.enterprisedb.com/downloads/postgres-postgresql-downloads

    ※ **Select Components** の画面では、Command Line Tools のみを選択して進めます。

7. 以下の URL より Azure Database for PostgreSQL の接続に必要となるルート証明書を任意のフォルダーにダウンロードします。
https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem

8. コマンドプロンプトを開いて以下を実行します。

※ホスト名やユーザー名の @ 以降、ルート証明書（pem ファイル）のパスはご自身の環境に併せて変更してから実行してください。
``` > C:\Program Files\PostgreSQL\13\bin\psql.exe "sslmode=verify-full sslrootcert=BaltimoreCyberTrustRoot.crt.pem host=xx-jpepsql.postgres.database.azure.com dbname=postgres user=azlabadmin@xx-jpepsql"

> CREATE DATABASE mypgsqldb;
> \c mypgsqldb

> CREATE TABLE inventory (
>    id serial PRIMARY KEY, 
>    name VARCHAR(50), 
>    quantity INTEGER
> );

> \dt

> INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
> INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);

> SELECT * FROM inventory;
```

## タスク 4: リソース マネージャー テンプレートを使用して複数のリソースを一括で展開する
このタスクでは、追加の Web サーバとアプリケーション ゲートウェイを展開します。
また、後半の演習で必要となる一部のリソースも一括で展開します。

今回は Azure の IaC (Infrastructure as a Code) ソリューションであるリソース マネージャー テンプレートを使用します。

1. 以下の [**Deploy to Azure**] ボタンをクリックして Azure ポータルを開き、Contoso アプリケーションの高可用性に必要となる、追加のインフラストラクチャを コンポーネントのテンプレート デプロイを起動します。プロンプトが表示された場合は、サブスクリプションの資格情報を使用して Azure ポータルにログインします。

[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ftamatsuxyz%2Fazurews%2Fmain%2FTemplates%2Fazuredeploy.json)

2. **カスタム デプロイ**画面で、以下の情報を入力します。入力後、[**確認および作成**] ボタンをクリックします。
    * リソース グループ: **Prefix**-azurews (既存）
    * リージョン: 東日本
    * Admin Username: azlabadmin
    * Admin Password: (**任意のパスワードを入力**)
3. リソースがデプロイされるのを待っている間、テンプレートの内容を確認してください。あなたは移動して、テンプレートを確認でき **Prefix**-azurews の選択、リソース・グループの展開リソースグループ内に続く、展開のいずれかを選択します。

テンプレートには、必要なさまざまなリソースを含む、以下の 5 つの子テンプレートが含まれていることに注目してください。
* 2 番目の Web サーバーとなる WebVM02（仮想マシン）
* Web サーバー用の負荷分散サービスである AppGW（アプリケーション ゲートウェイ）
* リージョン冗長化のための負荷分散サービスである Azure フロントドア
* セカンダリ リージョン用の仮想ネットワーク
* セカンダリ リージョン用の Web サーバーとなる WebVM10（仮想マシン）


4. **Prefix**-azurews リソース グループに移動し、左側のナビゲーションの [デプロイ] を選択することで、リソースのデプロイ状態を確認できます。次のタスクに進む前に、すべてのテンプレートの展開ステータスが成功であることを確認してください。
5. Azure portal の上部のナビゲーション バーで [Cloud Shell] をクリックして、[PowerShell] を選択します。
6. 次のコマンドを実行して、IIS を追加の Web サーバーにもインストールします。
 
    ※ 以下のコマンドの ResourceGroupName パラメーターをご自身の値に変更してから実行します。
```azurepowershell
 Set-AzVMExtension `
  -ResourceGroupName xx-azurews `
  -ExtensionName IIS `
  -VMName WebVM02 `
  -Publisher Microsoft.Compute `
  -ExtensionType CustomScriptExtension `
  -TypeHandlerVersion 1.4 `
  -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
  -Location JapanEast
```

```azurepowershell
 Set-AzVMExtension `
  -ResourceGroupName xx-azurews `
  -ExtensionName IIS `
  -VMName WebVM11 `
  -Publisher Microsoft.Compute `
  -ExtensionType CustomScriptExtension `
  -TypeHandlerVersion 1.4 `
  -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
  -Location JapanWest
```


## 演習 2: ネットワーク
---

## タスク 1: Azure Database for PostgreSQL をプライベート エンドポイントで保護する
Contoso 社からセキュリティ要件をヒアリングしたところ、Azure Database for PostgreSQL は、特定の仮想ネットワークからのみアクセスできるように制限をする必要があることがわかりました。
Azure の提供機能としては、サービス エンドポイントかプライベート エンドポイントが要件を満たすことを特定しましたが、
更なるヒアリングを行ったところ、他の Azure Database for PostgreSQL リソースへのアクセスを禁止する必要がわかりました。

Azure Database for PostgreSQL にプライベート エンドポイントを構成することで、そのサーバーを仮想ネットワーク (VNet) 内に取り入れることができます。
プライベート エンドポイントでは、VNet 内の他のリソースと同様に、データベース サーバーへの接続に使用できるサブネット内のプライベート IP が公開されるため、
他のデータベース サーバーへの接続を禁止しつつ、自信が所有するデータベース サーバーだけにアクセスを許可するという構成がとれます。

> [*注意*]
>
> プライベート リンク機能は、汎用またはメモリ最適化のいずれかの価格レベルの Azure Database for PostgreSQL サーバーでのみ使用可能です。

1. Azure ポータル上部の、検索欄に [Private Link] と入力し、表示された [Private Link] をクリックします。
2. [プライベート エンドポイントの作成] ボタンを選択します。
3. **基本**タブで以下の情報を入力します。入力後は、[次:リソース] をクリックします。
    * リソース グループ: **Prefix**-azurews (既存）
    * 名前: jpepsqlPrivateEndpoint
    * リージョン: 東日本
4. **リソース**タブで、以下の情報を入力します。入力後は、[次:構成] をクリックします。
    * リソースの種類: Microsoft.DBforPostgreSQL/servers
    * リソース: **Prefix**-jpepsql
    * ターゲット サブリソース: postgresqlServer
5. **構成**タブで以下の情報を入力します。入力後は、[**確認および作成**] をクリックします。
    * 仮想ネットワーク: jpevnet
    * サブネット: web
    * プライベート DNS ゾーンと統合する: はい
    * プライベート DNS ゾーン: （新規）privatelink.postgres.database.azure.com
6. **検証に成功しました** のメッセージが出力されたら、 [**作成**] ボタンをクリックします。
7. **デプロイが完了しました** のメッセージが出力されることを確認します。
8. WebVM01 に RDP 接続でログインします。（手順は、演習 1 - タスク 3 を参照してください）
9. コマンドプロンプトを開いて以下を実行します。

※ホスト名やユーザー名の @ 以降、ルート証明書（pem ファイル）のパスはご自身の環境に併せて変更してから実行してください。
```
> C:\Program Files\PostgreSQL\13\bin\psql.exe "sslmode=verify-full sslrootcert=BaltimoreCyberTrustRoot.crt.pem host=xx-jpepsql.postgres.database.azure.com dbname=postgres user=azlabadmin@xx-jpepsql"

> \dt

> SELECT * FROM inventory;
```

## 演習 3: 可用性とスケーラビリティ
---
この演習では、セカンダリ リージョンに同等の構成を追加することで、アプリケーションの可用性を向上させます。

## タスク 1  Azure Database for PostgreSQL をリージョン冗長構成変更にする

1. プライマリ サーバーとして使用する既存の Azure Database for PostgreSQL サーバー (**Prefix**-jpepsql) を選択します。
2. サーバー サイドバーで、 [設定] の [レプリケーション] を選択します。
3. [＋レプリカの追加] を選択します。
4. 以下の情報を入力します。入力後は、[**OK**] をクリックします。
    * サーバー名: **Prefix**-jpwpsql
    * 場所: 西日本
5. **デプロイが完了しました** のメッセージが出力されることを確認します。


> 注意
>
> Basic レベルのサーバーは、同じリージョンのレプリケーションのみをサポートしています。

> 重要
>
> プライマリ サーバーで障害が発生した場合、読み取りレプリカに自動的にフェールオーバーされることは**ありません**。
> プライマリとレプリカの間のレプリケーションを停止することにより、レプリカが再起動され、独立したスタンドアロンの読み取りと書き込みが可能なサーバーとして昇格します。
>
> また、プライマリ サーバーと読み取りレプリカへのレプリケーションを停止した後、それを元に戻すことはできません。 読み取りレプリカは、読み取りと書き込みの両方をサポートするスタンドアロン サーバーになります。スタンドアロン サーバーをもう一度レプリカにすることはできません。

6. 演習 2 - 2 と同様に、レプリカ環境に対しても、プライベート エンドポイントの設定を追加します。
ウィザードで入力する情報は、以下を参照してください。

* **基本**タブで以下の情報を入力します。
    * リソース グループ: **Prefix**-azurews (既存）
    * 名前: jpwpsqlPrivateEndpoint
    * リージョン: 東日本
* **リソース**タブで、以下の情報を入力します。
    * リソースの種類: Microsoft.DBforPostgreSQL/servers
    * リソース: **Prefix**-jpwpsql
    * ターゲット サブリソース: postgresqlServer
* **構成**タブで以下の情報を入力します。
    * 仮想ネットワーク: jpwvnet
    * サブネット: web
    * プライベート DNS ゾーンと統合する: はい
    * プライベート DNS ゾーン: （新規）privatelink.postgres.database.azure.com


## タスク 2: Web サーバーを Azure フロント ドアでリージョン冗長構成に変更する

