# **Ivanti製品が侵害された際のフォレンジック調査**

## **1\. 概要**

近年、Ivanti製品を標的としたセキュリティ侵害の頻度と深刻度が増加しています1。特に、インターネットに公開されているVPNなどの製品は、国家レベルの攻撃者を含む高度な脅威主体にとって価値の高い標的となっています。このような状況において、セキュリティ侵害が発生した場合に迅速かつ効果的に対応し、被害を最小限に抑えるためには、堅牢なフォレンジック調査能力が不可欠です。本レポートでは、侵害されたIvanti製品に対するフォレンジック調査を実施するための包括的なガイドラインを提供します。これには、過去のセキュリティ侵害事例、関連するログの種類と保存場所、収集すべきアーティファクト、役立つ可能性のあるツールと技術、影響を受けたシステムの特定と影響範囲の評価手順、復旧と再発防止策、調査結果の報告における重要なポイント、そして将来の侵害を防ぐための最新のセキュリティ情報と予防策が含まれます。

## **2\. はじめに**

Ivantiは、VPN、エンドポイント管理ツール、クラウドサービスアプライアンスなど、多岐にわたるIT管理およびセキュリティソリューションを提供しています。これらの製品は、企業のインフラストラクチャにおいて重要な役割を果たしており、その侵害は深刻な影響を及ぼす可能性があります。ネットワーク境界に位置するIvanti Connect Secureなどの製品が、脅威主体による標的となる傾向が強まっています1。ネットワーク境界デバイスへの攻撃は、組織の内部ネットワークへの侵入経路を確立することを目的としており、そのセキュリティ対策は極めて重要です。本レポートの目的は、侵害されたIvanti製品に対するフォレンジック調査を実施するための包括的な手順と情報を提供することです。

## **3\. Ivanti製品における過去のセキュリティ侵害事例と攻撃手法**

過去に発生したIvanti製品のセキュリティ侵害事例とその際に用いられた攻撃手法を理解することは、フォレンジック調査において極めて重要です。

### **3.1 Ivanti Connect SecureおよびPolicy Secureの脆弱性（2024年初頭）**

2024年初頭には、CVE-2023-46805（認証バイパス）とCVE-2024-21887（コマンドインジェクション）の脆弱性が悪用され、多くの場合、これらが組み合わされて認証なしでのリモートコード実行（RCE）が引き起こされました3。複数の脆弱性を連鎖的に悪用する攻撃手法は、攻撃の成功率を高めるために高度な攻撃者が用いるものであり、組織は複数の潜在的な弱点に対処する必要があることを示唆しています。この攻撃には、UNC5221（中国を拠点とするスパイ活動グループと疑われる）などの脅威主体が関与しており、Webシェル（GLASSTOKEN、LIGHTWIRE、CHAINLINEなど）、認証情報窃取ツール（WARPWIREなど）、およびバックドア（ZIPLINEなど）が展開されました3。特定の脅威主体のTTP（戦術、技術、手順）を理解することで、脅威ハンティングやインシデント対応の取り組みを大幅に向上させることができます。さらに、SAMLコンポーネントにおけるCVE-2024-21893（サーバーサイドリクエストフォージェリ）およびCVE-2024-22024（XML外部エンティティの脆弱性）の悪用も確認されています12。認証コンポーネントの脆弱性は、セキュリティ制御を迂回し、不正アクセスを可能にするため、特に深刻な影響を及ぼす可能性があります。

### **3.2 Ivanti Connect Secureの脆弱性（CVE-2025-22457、2025年初頭）**

2025年初頭には、UNC5221に起因する、リモートコード実行につながる重大なスタックベースのバッファオーバーフローの脆弱性CVE-2025-22457の悪用が確認されました1。当初は軽微な脆弱性と考えられていたにもかかわらず、高度な攻撃者はパッチをリバースエンジニアリングすることで新たな悪用方法を発見しました。これは、初期パッチ適用後も継続的な警戒が必要であることを強調しています。この攻撃では、TRAILBLAZEやBRUSHFIREといった新たなマルウェアファミリーがSPAWNエコシステムとともに展開されました8。攻撃者は、特定の脆弱性を標的とし、検知を回避するために常に新しいマルウェアを開発しています。

### **3.3 Ivanti Connect Secureの脆弱性（CVE-2025-0282、2025年初頭）**

同じく2025年初頭には、ゼロデイ脆弱性CVE-2025-0282が悪用され、バックドアがインストールされる事例が発生しました2。ゼロデイ脆弱性は、パッチが提供される前に悪用されるため、早期の検知と緩和が非常に重要です。この攻撃では、IvantiのIntegrity Checker Tool（ICT）が検知に利用されましたが、攻撃者によって改ざんが試みられたことも報告されています4。これは、ベンダー提供のツールだけに頼るのではなく、多層的な検知メカニズムが必要であることを示唆しています。

### **3.4 Ivanti Endpoint Managerの脆弱性（2025年初頭）**

2025年初頭には、複数の認証情報強制の脆弱性（CVE-2024-13159、CVE-2024-13160、CVE-2024-13161、CVE-2024-10811）が悪用され、サーバーが侵害される可能性が指摘されました5。エンドポイント管理ソリューションの脆弱性は、組織内の多数のエンドポイントへの広範なアクセスを攻撃者に提供する可能性があるため、特に注意が必要です。

### **3.5 Ivanti Cloud Service Appliance（CSA）の脆弱性（2024年後半）**

2024年後半には、CVE-2024-8963（管理バイパス）が他の脆弱性と組み合わされて悪用され、リモートコード実行や認証情報の窃取が行われました5。サポート終了製品も標的となる可能性があるため、常にサポートされている最新バージョンへのアップグレードが重要です。

| 影響を受ける製品 | 悪用されたCVE | 確認された脅威主体 | 攻撃手法の概要 |
| :---- | :---- | :---- | :---- |
| Connect Secure, Policy Secure | CVE-2023-46805, CVE-2024-21887 | UNC5221 | 認証バイパスとコマンドインジェクションの連鎖によるRCE |
| Connect Secure | CVE-2025-22457 | UNC5221 | スタックベースのバッファオーバーフローによるRCE |
| Connect Secure | CVE-2025-0282 | 不明 | ゼロデイ脆弱性を利用したバックドアの設置 |
| Endpoint Manager | CVE-2024-13159, CVE-2024-13160, CVE-2024-13161, CVE-2024-10811 | 不明 | 認証情報の強制によるサーバー侵害の可能性 |
| Cloud Service Appliance | CVE-2024-8963, その他 | 不明 | 管理バイパスと他の脆弱性の組み合わせによるRCE、認証情報窃取 |

**表3.1: 過去の主なIvanti製品侵害事例の概要**

## **4\. フォレンジック調査のためのIvanti製品ログの特定と活用**

侵害されたIvanti製品のフォレンジック調査において、ログは非常に重要な情報源となります。

### **4.1 Ivanti Connect SecureおよびPolicy Secureのログ**

Ivanti Connect SecureおよびPolicy Secureは、多岐にわたるログを生成し、これらはフォレンジック調査に役立ちます13。

* **イベントログ:** システムイベント、ユーザーアクセスイベント（ログイン/ログアウト、SAML/Javaアクセス）、管理者アクセスイベント（構成変更、管理者ログイン）、セキュリティイベントなどが記録されます。これらのログを分析することで、攻撃のタイムラインを再構築し、初期アクセス試行やシステム内での攻撃者の活動を追跡することができます。イベントログには、システムやユーザーの様々な活動に関する重要なタイムスタンプと詳細情報が含まれています。  
* **デバッグログ:** 詳細な低レベルのシステム情報が含まれており、トラブルシューティングに役立ちますが、悪用試行の詳細を明らかにする可能性もあります57。デバッグログは冗長な情報を含む可能性がありますが、通常のエラーメッセージやアクティビティパターンから逸脱する不審な活動の兆候が見られることがあります。  
* **アクセスログ:** ユーザーのアクセス試行とセッションの記録であり、不正アクセスを特定するのに役立ちます35。通常とは異なる時間帯やIPアドレスからのログイン試行、または連続したログイン失敗は、悪意のある活動を示唆している可能性があります。  
* **ポリシー追跡ログ:** 特定のユーザーに対するポリシー評価の詳細情報が含まれており、ポリシーがバイパスまたは操作されたかどうかを理解するのに役立ちます37。ポリシーの適用状況の変化や予期しないポリシーの適用は、侵害の兆候である可能性があります。  
* **クライアント側ログ:** ユーザーエンドポイント上のIvanti Secure Access Clientからのログであり、VPN接続の問題の追跡や侵害された可能性のあるエンドポイントの特定に役立ちます56。クライアント側のログは、ユーザーのVPN接続が乗っ取られたり、特定のエンドポイントから悪意のある活動が発生したりした場合に、その証拠を提供することができます。

### **4.2 Ivanti Endpoint Managerのログ**

Ivanti Endpoint Managerも、フォレンジック調査に役立つ様々なログを生成します41。

* **コンソールログ:** EPMコンソール内でのアクティビティが記録され、管理者の操作や不正な変更の可能性を追跡するのに役立ちます43。通常とは異なる管理者アクティビティ、例えば新しいアカウントの作成やポリシーの変更などは、侵害の兆候である可能性があります。  
* **エージェントログ:** 管理対象のエンドポイントからの詳細なログであり、マルウェアの展開、パッチ管理の問題、その他の不審なアクティビティの特定に不可欠です43。エージェントログは、EPMによって管理されている個々のエンドポイントで何が起こっているかについての可視性を提供します。  
* **サーバーログ:** スケジューリング、インベントリ、ソフトウェア配布などのコアEPMサーバープロセスに関連するログです43。サーバーログは、EPMサーバー自体が侵害され、マルウェアの配布に使用されたかどうかを特定するのに役立ちます。

### **4.3 Ivanti Cloud Service Appliance（CSA）のログ**

Ivanti Cloud Service Appliance（CSA）の具体的なログの詳細については、スニペットではあまり触れられていませんが、一般的なシステムログとアクセスログがフォレンジック調査に関連する可能性があります。

### **4.4 ログの保存場所と設定**

これらのログのデフォルトの保存場所は、ローカルファイルシステムまたはsyslogサーバーです43。フォレンジック調査の目的で詳細なログ記録を構成および有効にする方法を理解することが重要です43。ログを安全なsyslogサーバーに一元化することで、より効果的な分析が可能になり、攻撃者によるローカルログの改ざんを防ぐことができます56。これは、侵害の可能性のある状況では特に重要なセキュリティのベストプラクティスです。さらに、Ivanti Connect Secure Logs Parserなどのツールや技術を利用して、Ivantiログの解析と分析を効率化することができます35。専用ツールは、独自のログ形式の解析プロセスを大幅に簡素化することができます。

## **5\. Ivantiシステムにおけるフォレンジック調査のための重要なアーティファクト**

侵害されたIvantiシステムから収集すべき重要なアーティファクトを特定することは、フォレンジック調査において不可欠です。

* **メモリダンプ:** システムのメモリをキャプチャして、実行中のプロセス、インジェクションされたコードを分析し、削除された可能性のある情報を復元します3。メモリ分析は、ファイルシステム上では明らかにならない可能性のあるアクティブなマルウェアや攻撃者のテクニックを明らかにすることができます。マルウェアはディスクに書き込まずにメモリ内で実行されることが多いため、メモリ分析は不可欠です。  
* **ファイルシステムイメージ:** システムのファイルシステムの完全なイメージを作成して、オフライン分析を行います。これには、ファイルのタイムスタンプの調査、新しく作成または変更されたファイルの特定、削除されたファイルの復元が含まれます9。ファイルシステム分析は、ドロップされたマルウェア、バックドア、およびその他の悪意のあるコンポーネントを明らかにすることができます。攻撃者は証拠隠蔽を試みても、多くの場合、システム上にファイルを残します。  
* **ネットワーク接続情報:** アクティブなネットワーク接続と最近のネットワーク接続に関するデータを収集して、コマンドアンドコントロール（C2）サーバーまたはその他の悪意のあるインフラストラクチャとの通信を特定します3。ネットワーク接続は、攻撃者の通信経路とデータ漏洩の範囲を明らかにすることができます。マルウェアは、コマンドを受信したり、盗まれたデータを送信したりするために、外部サーバーと通信することがよくあります。  
* **実行中のプロセス:** 現在実行中のすべてのプロセスをリストアップして、悪意のあるまたは不明なプロセスを特定します8。不正なプロセスの特定は、アクティブなマルウェアを検出するための重要なステップです。マルウェアは通常、侵害されたシステム上でプロセスとして実行されます。  
* **スケジュールされたタスク:** スケジュールされたタスクを調べて、自動的に実行されるように設定された悪意のあるスクリプトやプログラムがないか確認します43。攻撃者は永続化のためにスケジュールされたタスクを使用することがよくあります。スケジュールされたタスクを使用すると、ユーザーの操作なしに、特定の時間または間隔でプログラムを実行できます。  
* **レジストリエントリ（WindowsベースのIvanti製品、例：Endpoint Manager）:** 特定のレジストリキーは、マルウェアの永続化メカニズムや攻撃者によって行われた構成変更を明らかにすることができます43。Windowsレジストリは、マルウェアが永続化を確立するための一般的な場所です。マルウェアは、システム再起動後に実行されるように、レジストリキーを変更することがよくあります。  
* **構成ファイル:** Ivanti製品の構成ファイルを収集して、不正な変更、たとえばアクセス制御リストの変更や有効になっている機能などを確認します15。攻撃者は、アクセスを容易にしたり、検出を回避したりするために構成を変更する可能性があります。構成ファイルは、Ivanti製品の動作方法を制御します。  
* **Webサーバーログ（該当する場合）:** Webインターフェイスを備えたIvanti製品の場合、Webサーバーログを調べて、不審なリクエストやアクセスパターンがないか確認します13。Webサーバーログは、悪用試行や悪意のあるWebシェルへのアクセスを記録することができます。多くのIvanti製品には、攻撃者が標的とする可能性のあるWebベースのインターフェイスがあります。  
* **Integrity Checker Tool（ICT）の出力:** その信頼性は損なわれている可能性がありますが、ICTの出力は、ファイル変更の最初の兆候を提供することができます4。侵害が疑われる前後のICT出力を比較することで、変更点を強調表示できます。ICTはファイル変更を検出するように設計されています。

## **6\. Ivanti製品分析に役立つフォレンジックツールと技術**

Ivanti製品の侵害調査に役立つ可能性のある一般的なフォレンジックツールと技術を以下に示します。

* **メモリ分析ツール:** Volatility、Rekall 19。  
* **ディスクイメージングツール:** FTK Imager、EnCase。  
* **ログ分析ツール:** Splunk、ELK Stack、専用パーサー（例：Ivanti Connect Secure Logs Parser）35。  
* **ネットワーク分析ツール:** Wireshark、tcpdump 57。  
* **Endpoint Detection and Response（EDR）ソリューション:** 多くのEDRツールは、Ivanti製品によって管理されるエンドポイント上の脅威を検出および分析する機能を持っています36。

Ivanti固有のツールと技術には以下が含まれます。

* **Ivanti Integrity Checker Tool（ICT）:** 高度な侵害を検出する上での目的と限界について説明します4。  
* **Ivanti Neurons for RBVM（リスクベース脆弱性管理）:** 厳密にはフォレンジックツールではありませんが、悪用された可能性のある脆弱性を特定し、優先順位を付けるのに役立ちます75。  
* **Ivanti Security Controls:** パッチ管理やその他のセキュリティ機能を提供し、環境のセキュリティ体制を理解する上で関連性があります66。

ファイルハッシュの比較、既知の悪意のある指標（IOC）のネットワークトラフィックの分析、およびシステムログの異常の調査などの技術について説明します3。包括的な調査には、様々なツールと技術を組み合わせた多面的なアプローチが必要です。単一のツールや方法に頼るだけでは、重要な証拠を見逃す可能性があります。

## **7\. 侵害されたIvantiシステムの特定と影響範囲の特定手順**

侵害された可能性のあるIvantiシステムを特定し、影響範囲を評価するための手順を以下に示します。

* **セキュリティアラートとアドバイザリの確認:** Ivantiおよびサイバーセキュリティ機関からの最新の脆弱性および悪用レポートについて常に情報を把握します9。  
* **IvantiのIntegrity Checker Tool（ICT）の実行:** その限界を理解し、初期評価ツールとして使用します4。  
* **ログの不審なアクティビティの分析:** 通常とは異なるログイン試行、予期しないトラフィック、または既知の脆弱性に関連するエラーメッセージを探します13。  
* **既知の侵害指標（IOC）の確認:** Ivanti攻撃に関連するIOC（例：IPアドレス、ドメイン、ファイルハッシュ）を利用します7。  
* **ネットワークトラフィックの異常の監視:** Ivantiシステムを発信元または宛先とする通常とは異なる通信パターンを探します3。

影響範囲を評価する方法は次のとおりです。

* **影響を受けるシステムの特定:** 組織内に展開されているIvanti製品を特定します。  
* **ネットワークセグメンテーションの分析:** 潜在的な水平展開を評価するために、ネットワークアーキテクチャを理解します3。  
* **ユーザーアカウントと認証情報の調査:** Ivanti製品を介してアクセスされた可能性のある侵害されたユーザーアカウントがないか確認します3。  
* **水平展開の兆候の確認:** Ivantiデバイスから発生した可能性のある侵害の兆候がないか、ネットワーク上の他のシステムを調査します3。  
* **データ漏洩分析:** 侵害されたIvantiシステムまたは内部ネットワークからデータが盗まれた証拠がないか確認します3。

## **8\. 侵害されたIvantiシステムの復旧と再発防止策**

侵害されたIvantiシステムを復旧し、将来のインシデントを防止するための戦略を以下に詳述します。

侵害されたIvantiシステムを復旧するための手順：

* **影響を受けるシステムの隔離:** 侵害されたIvantiシステムを直ちにネットワークから切断し、さらなる損害や水平展開を防ぎます3。  
* **工場出荷時設定へのリセットの実行:** 多くのIvanti製品では、永続的なマルウェアを排除するために工場出荷時設定へのリセットが推奨されます9。  
* **既知のクリーンバックアップからの再イメージング:** 利用可能な場合は、侵害前に作成されたバックアップからシステムを復元します26。  
* **パッチとアップデートの適用:** Ivantiがリリースした最新のセキュリティパッチとアップデートを直ちに適用します3。  
* **最新のサポート対象バージョンへのアップグレード:** 侵害されたシステムが旧式またはサポート終了バージョンを実行している場合は、最新のサポート対象バージョンにアップグレードします3。  
* **パスワードの変更と証明書の失効:** 侵害されたシステムに関連するすべてのパスワードをリセットし、侵害された可能性のある証明書を失効させます9。  
* **徹底的な監視の実施:** 復旧後、再感染または永続的な悪意のある活動の兆候がないかシステムを綿密に監視します3。

将来のインシデントを最小限に抑えるための予防措置：

* **堅牢なパッチ管理プログラムの実装:** すべてのIvanti製品およびその他のソフトウェアのタイムリーなパッチ適用を保証します3。  
* **多要素認証（MFA）の強制:** すべての管理者アカウントとユーザーアカウント、特にVPNアクセスに対してMFAを有効にします3。  
* **ネットワークセグメンテーションの実施:** 潜在的な侵害の影響を制限するために、ネットワークセグメンテーションを実施します3。  
* **脆弱性スキャンの定期的な更新と実行:** 脆弱性スキャンツールを使用して、潜在的な弱点を特定し、対処します3。  
* **強力なパスワードポリシーの実装:** 複雑なパスワードと定期的なパスワード変更を強制します62。  
* **不要なサービスと機能の無効化:** Ivanti製品で未使用のサービスまたは機能を無効にして、攻撃対象領域を縮小します73。  
* **アクセス制御リスト（ACL）の実装:** 管理インターフェイスと機密リソースへのアクセスを制限します28。  
* **ログ記録の有効化と監視:** 包括的なログ記録が有効になっていることを確認し、ログを定期的に監視して不審なアクティビティがないか確認します13。  
* **定期的なセキュリティ監査と侵入テスト:** 潜在的な脆弱性を特定して対処するために、定期的なセキュリティ評価を実施します3。  
* **従業員のセキュリティ意識向上トレーニング:** Ivanti VPN認証情報を標的とする可能性のあるフィッシングやその他のソーシャルエンジニアリング攻撃についてユーザーを教育します3。

## **9\. Ivantiセキュリティインシデントのフォレンジック調査結果報告における重要な考慮事項**

フォレンジック調査中にすべての調査結果を文書化することの重要性を強調します。インシデントのタイムライン、特定された攻撃ベクトル、影響を受けたシステム、および侵害の範囲に関する詳細を含めます。収集されたすべてのアーティファクト、および分析に使用されたツールと技術を文書化します。可能な場合は、調査結果の明確かつ簡潔な概要（属性を含む）を提供します。修復および予防措置に関する推奨事項を含めます。改ざんされたログや暗号化されたデータなど、調査中に遭遇した制限事項や課題を強調します。レポートの対象読者（技術者向けか経営層向けか）を考慮し、それに応じて言語と詳細レベルを調整します。

## **10\. 今後の対策：最新のIvantiセキュリティ情報と予防策**

最新の脆弱性およびパッチに関する情報については、Ivantiの公式セキュリティアドバイザリおよびIvanti Trust Centerを参照するようにユーザーに指示します9。Ivanti製品に関するCISAおよびその他のサイバーセキュリティ機関のアラートと推奨事項に従うことを推奨します9。新たな脅威を検出し、対応するために、継続的な監視と脅威ハンティング活動の重要性を強調します3。製品セキュリティの強化に対するIvantiの取り組みと、ベストプラクティスに関する推奨事項を強調します39。

## **11\. 結論**

Ivanti製品が企業インフラストラクチャにおいて果たす重要な役割と、それらが直面している持続的な脅威を改めて述べます。潜在的な侵害のリスクと影響を軽減するためには、プロアクティブなセキュリティ対策と効果的なフォレンジック調査能力が不可欠であることを再強調します。組織は、Ivanti製品に関連する最新のセキュリティ脅威とベストプラクティスについて常に情報を把握しておく必要があります。セキュリティを優先し、推奨される対策を実施し、必要に応じて徹底的なフォレンジック調査を実施する準備をすることが最終的な行動喚起となります。

#### **引用文献**

1. Is Ivanti the problem or a symptom of a systemic issue with network devices? \- CyberScoop, 4月 22, 2025にアクセス、 [https://cyberscoop.com/ivanti-exploited-vulnerabilities-network-edge-devices-kev-list/](https://cyberscoop.com/ivanti-exploited-vulnerabilities-network-edge-devices-kev-list/)  
2. Attackers lodge backdoors into Ivanti Connect Secure devices \- Cybersecurity Dive, 4月 22, 2025にアクセス、 [https://www.cybersecuritydive.com/news/ivanti-connect-secure-backdoors/738252/](https://www.cybersecuritydive.com/news/ivanti-connect-secure-backdoors/738252/)  
3. Ivanti VPN Vulnerability: What You Need to Know \- Palo Alto Networks, 4月 22, 2025にアクセス、 [https://www.paloaltonetworks.com/cyberpedia/ivanti-VPN-vulnerability-what-you-need-to-know](https://www.paloaltonetworks.com/cyberpedia/ivanti-VPN-vulnerability-what-you-need-to-know)  
4. Ivanti Warns of New Zero-Day Attacks Hitting Connect Secure Product \- SecurityWeek, 4月 22, 2025にアクセス、 [https://www.securityweek.com/ivanti-warns-of-new-zero-day-attacks-hitting-connect-secure-product/](https://www.securityweek.com/ivanti-warns-of-new-zero-day-attacks-hitting-connect-secure-product/)  
5. Ivanti up against another attack spree as hackers target its endpoint manager, 4月 22, 2025にアクセス、 [https://www.cybersecuritydive.com/news/ivanti-endpoint-manager-hackers-attack/728814/](https://www.cybersecuritydive.com/news/ivanti-endpoint-manager-hackers-attack/728814/)  
6. China-backed espionage group hits Ivanti customers again \- CyberScoop, 4月 22, 2025にアクセス、 [https://cyberscoop.com/china-espionage-group-ivanti-vulnerability-exploits/](https://cyberscoop.com/china-espionage-group-ivanti-vulnerability-exploits/)  
7. Zero-day vulnerability in Ivanti Connect Secure exploited for weeks \- Field Effect, 4月 22, 2025にアクセス、 [https://fieldeffect.com/blog/zero-day-ivanti-connect-secure-exploited-for-weeks](https://fieldeffect.com/blog/zero-day-ivanti-connect-secure-exploited-for-weeks)  
8. UNC5221's Latest Exploit: Weaponizing CVE-2025-22457 in Ivanti Connect Secure, 4月 22, 2025にアクセス、 [https://www.picussecurity.com/resource/blog/unc5221-cve-2025-22457-ivanti-connect-secure](https://www.picussecurity.com/resource/blog/unc5221-cve-2025-22457-ivanti-connect-secure)  
9. Ivanti Connect Secure, Policy Secure, ZTA Gateways Flaw Under Active Exploitation, 4月 22, 2025にアクセス、 [https://www.hipaajournal.com/ivanti-connect-secure-active-exploitation/](https://www.hipaajournal.com/ivanti-connect-secure-active-exploitation/)  
10. Wisdomajoku/Ivanti-VPN-Attack-Case-Study \- GitHub, 4月 22, 2025にアクセス、 [https://github.com/WisdomAjoku7/Ivanti-VPN-Attack-Case-Study](https://github.com/WisdomAjoku7/Ivanti-VPN-Attack-Case-Study)  
11. A Brief Analysis of the Ivanti VPN Breach Affecting CISA \- TraceSecurity, 4月 22, 2025にアクセス、 [https://www.tracesecurity.com/blog/news/a-brief-analysis-of-the-ivanti-vpn-breach-affecting-cisa](https://www.tracesecurity.com/blog/news/a-brief-analysis-of-the-ivanti-vpn-breach-affecting-cisa)  
12. Threat Actors Exploit Multiple Vulnerabilities in Ivanti Connect ... \- CISA, 4月 22, 2025にアクセス、 [https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b)  
13. Darktrace's Early Detection of the Latest Ivanti Exploits, 4月 22, 2025にアクセス、 [https://darktrace.com/blog/darktraces-early-detection-of-the-latest-ivanti-exploits](https://darktrace.com/blog/darktraces-early-detection-of-the-latest-ivanti-exploits)  
14. Exploitation of vulnerabilities affecting Ivanti Connect... \- NCSC.GOV.UK, 4月 22, 2025にアクセス、 [https://www.ncsc.gov.uk/news/exploitation-ivanti-vulnerabilities](https://www.ncsc.gov.uk/news/exploitation-ivanti-vulnerabilities)  
15. Post-Exploitation Activities of Ivanti CS/PS Appliances \- Darktrace, 4月 22, 2025にアクセス、 [https://darktrace.com/blog/the-unknown-unknowns-post-exploitation-activities-of-ivanti-cs-ps-appliances](https://darktrace.com/blog/the-unknown-unknowns-post-exploitation-activities-of-ivanti-cs-ps-appliances)  
16. Cutting Edge, Part 3: Investigating Ivanti Connect Secure VPN Exploitation and Persistence Attempts | Google Cloud Blog, 4月 22, 2025にアクセス、 [https://cloud.google.com/blog/topics/threat-intelligence/investigating-ivanti-exploitation-persistence/](https://cloud.google.com/blog/topics/threat-intelligence/investigating-ivanti-exploitation-persistence/)  
17. Threat Actors Exploit Multiple Vulnerabilities in Ivanti Connect Secure and Policy Secure Gateways | Cyber.gov.au, 4月 22, 2025にアクセス、 [https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories/threat-actors-exploit-multiple-vulnerabilities-ivanti-connect-secure-and-policy-secure-gateways](https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories/threat-actors-exploit-multiple-vulnerabilities-ivanti-connect-secure-and-policy-secure-gateways)  
18. Active Exploitation of Two Zero-Day Vulnerabilities in Ivanti Connect Secure VPN | Volexity, 4月 22, 2025にアクセス、 [https://www.volexity.com/blog/2024/01/10/active-exploitation-of-two-zero-day-vulnerabilities-in-ivanti-connect-secure-vpn/](https://www.volexity.com/blog/2024/01/10/active-exploitation-of-two-zero-day-vulnerabilities-in-ivanti-connect-secure-vpn/)  
19. How Memory Forensics Revealed Exploitation of Ivanti Connect Secure VPN Zero-Day Vulnerabilities | Volexity, 4月 22, 2025にアクセス、 [https://www.volexity.com/blog/2024/02/01/how-memory-forensics-revealed-exploitation-of-ivanti-connect-secure-vpn-zero-day-vulnerabilities/](https://www.volexity.com/blog/2024/02/01/how-memory-forensics-revealed-exploitation-of-ivanti-connect-secure-vpn-zero-day-vulnerabilities/)  
20. THREAT ALERT: Ivanti Connect Secure VPN Zero-Day Exploitation \- Cybereason, 4月 22, 2025にアクセス、 [https://www.cybereason.com/blog/threat-alert-ivanti-connect-secure-vpn-zero-day-exploitation](https://www.cybereason.com/blog/threat-alert-ivanti-connect-secure-vpn-zero-day-exploitation)  
21. Widespread exploitation of recently disclosed Ivanti vulnerabilities \- IBM, 4月 22, 2025にアクセス、 [https://www.ibm.com/think/x-force/exploitation-of-exposed-ivanti-vulnerabilities](https://www.ibm.com/think/x-force/exploitation-of-exposed-ivanti-vulnerabilities)  
22. Investigating a possible Ivanti compromise \- Northwave Cyber Security, 4月 22, 2025にアクセス、 [https://northwave-cybersecurity.com/whitepapers-articles/investigating-a-possible-ivanti-compromise](https://northwave-cybersecurity.com/whitepapers-articles/investigating-a-possible-ivanti-compromise)  
23. Cutting Edge, Part 2: Investigating Ivanti Connect Secure VPN Zero-Day Exploitation, 4月 22, 2025にアクセス、 [https://cloud.google.com/blog/topics/threat-intelligence/investigating-ivanti-zero-day-exploitation/](https://cloud.google.com/blog/topics/threat-intelligence/investigating-ivanti-zero-day-exploitation/)  
24. Suspected China-Nexus Threat Actor Actively Exploiting Critical Ivanti Connect Secure Vulnerability (CVE-2025-22457) | Google Cloud Blog, 4月 22, 2025にアクセス、 [https://cloud.google.com/blog/topics/threat-intelligence/china-nexus-exploiting-critical-ivanti-vulnerability](https://cloud.google.com/blog/topics/threat-intelligence/china-nexus-exploiting-critical-ivanti-vulnerability)  
25. Ivanti Connect Secure VPN Targeted in New Zero-Day Exploitation ..., 4月 22, 2025にアクセス、 [https://cloud.google.com/blog/topics/threat-intelligence/ivanti-connect-secure-vpn-zero-day/](https://cloud.google.com/blog/topics/threat-intelligence/ivanti-connect-secure-vpn-zero-day/)  
26. CISA warns new malware targeting Ivanti zero-day vulnerability | Cybersecurity Dive, 4月 22, 2025にアクセス、 [https://www.cybersecuritydive.com/news/cisa-warns-malware-targeting-ivanti-zero-day/743967/](https://www.cybersecuritydive.com/news/cisa-warns-malware-targeting-ivanti-zero-day/743967/)  
27. CVE-2025-22457: Critical Ivanti Vulnerability Details \- Truesec, 4月 22, 2025にアクセス、 [https://www.truesec.com/hub/blog/cve-2025-22457-ivanti-buffer-overflow-vulnerability](https://www.truesec.com/hub/blog/cve-2025-22457-ivanti-buffer-overflow-vulnerability)  
28. Security Advisory: CVE-2025-22457 – Critical Ivanti Flaw Actively Exploited to Deploy TRAILBLAZE and BRUSHFIRE Malware \- Insights | Integrity360, 4月 22, 2025にアクセス、 [https://insights.integrity360.com/security-advisory-cve-2025-22457-critical-ivanti-flaw-actively-exploited-to-deploy-trailblaze-and-brushfire-malware](https://insights.integrity360.com/security-advisory-cve-2025-22457-critical-ivanti-flaw-actively-exploited-to-deploy-trailblaze-and-brushfire-malware)  
29. Ivanti Connect Secure CVE-2025-22457 exploited in the wild | Rapid7 Blog, 4月 22, 2025にアクセス、 [https://www.rapid7.com/blog/post/2025/04/03/etr-ivanti-connect-secure-cve-2025-22457-exploited-in-the-wild/](https://www.rapid7.com/blog/post/2025/04/03/etr-ivanti-connect-secure-cve-2025-22457-exploited-in-the-wild/)  
30. CVE-2025-22457 – Critical RCE Bug in Multiple Ivanti Product \- Lumifi Cyber, 4月 22, 2025にアクセス、 [https://www.lumificyber.com/threat-library/cve-2025-22457-critical-rce-bug-in-multiple-ivanti-product/](https://www.lumificyber.com/threat-library/cve-2025-22457-critical-rce-bug-in-multiple-ivanti-product/)  
31. April Security Advisory Ivanti Connect Secure, Policy Secure & ZTA ..., 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/April-Security-Advisory-Ivanti-Connect-Secure-Policy-Secure-ZTA-Gateways-CVE-2025-22457?language=en\_US](https://forums.ivanti.com/s/article/April-Security-Advisory-Ivanti-Connect-Secure-Policy-Secure-ZTA-Gateways-CVE-2025-22457?language=en_US)  
32. A Vulnerability in Ivanti Products Could Allow for Remote Code Execution, 4月 22, 2025にアクセス、 [https://www.cisecurity.org/advisory/a-vulnerability-in-ivanti-products-could-allow-for-remote-code-execution\_2025-034](https://www.cisecurity.org/advisory/a-vulnerability-in-ivanti-products-could-allow-for-remote-code-execution_2025-034)  
33. Over 5K Ivanti VPNs vulnerable to critical bug under attack | Cybersecurity Dive, 4月 22, 2025にアクセス、 [https://www.cybersecuritydive.com/news/5k-ivanti-vpns-vulnerable-critical-flaw-under-attack/744748/](https://www.cybersecuritydive.com/news/5k-ivanti-vpns-vulnerable-critical-flaw-under-attack/744748/)  
34. Active exploitation of vulnerability affecting Ivanti... \- NCSC.GOV.UK, 4月 22, 2025にアクセス、 [https://www.ncsc.gov.uk/news/active-exploitation-ivanti-vulnerability](https://www.ncsc.gov.uk/news/active-exploitation-ivanti-vulnerability)  
35. Hexastrike/Ivanti-Connect-Secure-Logs-Parser \- GitHub, 4月 22, 2025にアクセス、 [https://github.com/Hexastrike/Ivanti-Connect-Secure-Logs-Parser](https://github.com/Hexastrike/Ivanti-Connect-Secure-Logs-Parser)  
36. Ivanti Exploited: Inside Active Threats & Risks \- Vulnerability \- Cyble, 4月 22, 2025にアクセス、 [https://cyble.com/blog/ivanti-exploited-vulnerabilites/](https://cyble.com/blog/ivanti-exploited-vulnerabilites/)  
37. SeizeCyber/Ivanti-Secure-Connect-Logs-Parser \- GitHub, 4月 22, 2025にアクセス、 [https://github.com/SeizeCyber/Ivanti-Secure-Connect-Logs-Parser](https://github.com/SeizeCyber/Ivanti-Secure-Connect-Logs-Parser)  
38. CVE-2025-22457: Ivanti Remote Code Execution Vulnerability Explained \- Picus Security, 4月 22, 2025にアクセス、 [https://www.picussecurity.com/resource/blog/cve-2025-22457-ivanti-remote-code-execution-vulnerability](https://www.picussecurity.com/resource/blog/cve-2025-22457-ivanti-remote-code-execution-vulnerability)  
39. Key FAQs Related to Ivanti Connect Secure, Policy Secure and ZTA Gateway Vulnerabilities, 4月 22, 2025にアクセス、 [https://www.ivanti.com/blog/key-faqs-related-to-ivanti-connect-secure-policy-secure-and-zta-gateway-vulnerabilities](https://www.ivanti.com/blog/key-faqs-related-to-ivanti-connect-secure-policy-secure-and-zta-gateway-vulnerabilities)  
40. Proof-of-concept exploit released for 4 Ivanti vulnerabilities | Cybersecurity Dive, 4月 22, 2025にアクセス、 [https://www.cybersecuritydive.com/news/proof-of-concept-exploit-released-for-4-ivanti-vulnerabilities/740475/](https://www.cybersecuritydive.com/news/proof-of-concept-exploit-released-for-4-ivanti-vulnerabilities/740475/)  
41. Ivanti Endpoint Manager Vulnerabilities: Critical CVEs & Exploit Details \- Horizon3.ai, 4月 22, 2025にアクセス、 [https://horizon3.ai/attack-research/attack-blogs/ivanti-endpoint-manager-multiple-credential-coercion-vulnerabilities/](https://horizon3.ai/attack-research/attack-blogs/ivanti-endpoint-manager-multiple-credential-coercion-vulnerabilities/)  
42. CISA: 3 Ivanti endpoint vulnerabilities exploited in the wild | Cybersecurity Dive, 4月 22, 2025にアクセス、 [https://www.cybersecuritydive.com/news/cisa-3-ivanti-endpoint-vulnerabilities-exploited-in-the-wild/742168/](https://www.cybersecuritydive.com/news/cisa-3-ivanti-endpoint-vulnerabilities-exploited-in-the-wild/742168/)  
43. End of Life Notice for Avalanche versions prior to 5.3.x, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/Endpoint-Manager-Additional-Logging-Options](https://forums.ivanti.com/s/article/Endpoint-Manager-Additional-Logging-Options)  
44. How to Easily Gather Ivanti EPM Logs From a Windows Client Device (PowerShell Script), 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/How-to-Easily-Gather-Ivanti-EPM-Logs-From-a-Windows-Client-Device-PowerShell-Script?ui-force-components-controllers-recordGlobalValueProvider.RecordGvp.getRecord=1](https://forums.ivanti.com/s/article/How-to-Easily-Gather-Ivanti-EPM-Logs-From-a-Windows-Client-Device-PowerShell-Script?ui-force-components-controllers-recordGlobalValueProvider.RecordGvp.getRecord=1)  
45. EBA Log Locations \- Ivanti Community, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/EBA-Log-Locations?ui-force-components-controllers-recordGlobalValueProvider.RecordGvp.getRecord=1](https://forums.ivanti.com/s/article/EBA-Log-Locations?ui-force-components-controllers-recordGlobalValueProvider.RecordGvp.getRecord=1)  
46. Agent Troubleshooting: Log Collection \- YouTube, 4月 22, 2025にアクセス、 [https://www.youtube.com/watch?v=\_0KUiGAohAI](https://www.youtube.com/watch?v=_0KUiGAohAI)  
47. How to Retrieve Diagnostic Log Files From a Windows Client in EPM \- Ivanti Community, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/How-to-retrieve-log-files-from-an-EPM-managed-client-machine-running-Ivanti-Endpoint-Manager-EPM-Agent-for-Windows](https://forums.ivanti.com/s/article/How-to-retrieve-log-files-from-an-EPM-managed-client-machine-running-Ivanti-Endpoint-Manager-EPM-Agent-for-Windows)  
48. EPM Logs in Managementsuite\\log Folder Are No Longer Getting Created or Updated, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/EPM-Logs-in-Managementsuite-log-Folder-Are-No-Longer-Getting-Created](https://forums.ivanti.com/s/article/EPM-Logs-in-Managementsuite-log-Folder-Are-No-Longer-Getting-Created)  
49. Viewing device diagnostics and logs \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ld/help/en\_US/LDMS/10.0/Windows/console-o-diagslogs.htm](https://help.ivanti.com/ld/help/en_US/LDMS/10.0/Windows/console-o-diagslogs.htm)  
50. Credential Coercion Vulnerabilities \- Ivanti Endpoint Manager \- Indusface, 4月 22, 2025にアクセス、 [https://www.indusface.com/blog/credential-coercion-vulnerabilities/](https://www.indusface.com/blog/credential-coercion-vulnerabilities/)  
51. Vulnerability in Ivanti Endpoint Manager Mobile Could Allow for Unauthorized Access to API Paths | Cyber Florida at USF, 4月 22, 2025にアクセス、 [https://cyberflorida.org/vulnerability-in-ivanti-endpoint-manager-mobile-could-allow-for-unauthorized-access-to-api-paths/](https://cyberflorida.org/vulnerability-in-ivanti-endpoint-manager-mobile-could-allow-for-unauthorized-access-to-api-paths/)  
52. Ivanti Releases Security Updates for Multiple Products \- CISA, 4月 22, 2025にアクセス、 [https://www.cisa.gov/news-events/alerts/2025/01/14/ivanti-releases-security-updates-multiple-products](https://www.cisa.gov/news-events/alerts/2025/01/14/ivanti-releases-security-updates-multiple-products)  
53. Threat Actors Chained Vulnerabilities in Ivanti Cloud Service Applications \- CISA, 4月 22, 2025にアクセス、 [https://www.cisa.gov/news-events/cybersecurity-advisories/aa25-022a](https://www.cisa.gov/news-events/cybersecurity-advisories/aa25-022a)  
54. Threat Actors Chained Vulnerabilities in Ivanti Cloud Service Applications, 4月 22, 2025にアクセス、 [https://www.ic3.gov/CSA/2025/250122.pdf](https://www.ic3.gov/CSA/2025/250122.pdf)  
55. FBI/CISA Share Details on Ivanti Exploits Chains: What Network Defenders Need to Know, 4月 22, 2025にアクセス、 [https://www.securityweek.com/fbi-cisa-share-details-on-ivanti-exploits-chains-what-network-defenders-need-to-know/](https://www.securityweek.com/fbi-cisa-share-details-on-ivanti-exploits-chains-what-network-defenders-need-to-know/)  
56. Logging and Monitoring \- Product Documentation \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_US/ICS/9.1RX/ag-9.1R17/logging-n-monitoring.htm](https://help.ivanti.com/ps/help/en_US/ICS/9.1RX/ag-9.1R17/logging-n-monitoring.htm)  
57. List of the logs and the steps to collect them for the Terminal Services issues, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/List-of-the-logs-and-the-steps-to-collect-them-for-the-Terminal-Services-issues?](https://forums.ivanti.com/s/article/List-of-the-logs-and-the-steps-to-collect-them-for-the-Terminal-Services-issues)  
58. Troubleshooting Tools \- Product Documentation \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_US/ICS/9.1RX/ag-9.1R17/troubleshooting-tools.htm](https://help.ivanti.com/ps/help/en_US/ICS/9.1RX/ag-9.1R17/troubleshooting-tools.htm)  
59. Using Policy Tracing and Debug Logs \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_US/ICS/vNow/INTmdmsdg/using\_policy\_tracing\_n.htm](https://help.ivanti.com/ps/help/en_US/ICS/vNow/INTmdmsdg/using_policy_tracing_n.htm)  
60. Logging and Monitoring \- Product Documentation \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_US/ICS/21.9R1/ag/logging\_n\_monitoring.htm](https://help.ivanti.com/ps/help/en_US/ICS/21.9R1/ag/logging_n_monitoring.htm)  
61. Ivanti Connect Secure \- Cortex Marketplace, 4月 22, 2025にアクセス、 [https://cortex.marketplace.pan.dev/marketplace/details/IvantiConnectSecure/](https://cortex.marketplace.pan.dev/marketplace/details/IvantiConnectSecure/)  
62. Ivanti ISM logs, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/question/0D5Do00000uowIVKAY/ivanti-ism-logs?language=en\_US](https://forums.ivanti.com/s/question/0D5Do00000uowIVKAY/ivanti-ism-logs?language=en_US)  
63. Steps to collect Desktop ISAC (Ivanti Secure Access Client) logs, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/KB17327?language=en\_US](https://forums.ivanti.com/s/article/KB17327?language=en_US)  
64. Ivanti Secure Access Client Log Files, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_US/ISAC/22.X/ag-22.X/log\_files.htm](https://help.ivanti.com/ps/help/en_US/ISAC/22.X/ag-22.X/log_files.htm)  
65. Uploading Ivanti Secure Access Client Log Files, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_US/ISAC/22.X/ag-22.X/uploading\_log\_files.htm](https://help.ivanti.com/ps/help/en_US/ISAC/22.X/ag-22.X/uploading_log_files.htm)  
66. How To: Gather console and target machine logging for Ivanti Security Controls, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/How-To-Gather-console-and-target-machine-logging-for-Ivanti-Security-Controls](https://forums.ivanti.com/s/article/How-To-Gather-console-and-target-machine-logging-for-Ivanti-Security-Controls)  
67. forums.ivanti.com, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/How-To-Gather-console-and-target-machine-logging-for-Ivanti-Security-Controls?language=en\_US\#:\~:text=Ivanti%20Security%20Controls%20installation%20logging%3A\&text=The%20setup%20and%20installation%20logs,%5CAppData%5CLocal%5CTemp.](https://forums.ivanti.com/s/article/How-To-Gather-console-and-target-machine-logging-for-Ivanti-Security-Controls?language=en_US#:~:text=Ivanti%20Security%20Controls%20installation%20logging%3A&text=The%20setup%20and%20installation%20logs,%5CAppData%5CLocal%5CTemp.)  
68. Listing and Purpose of Each Log File Generated by Ivanti Endpoint Security (Heat EMSS), 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/Listing-and-Purpose-of-Each-Log-File-Generated-by-Ivanti-Endpoint-Security-Heat-EMSS?nocache=https%3A%2F%2Fforums.ivanti.com%2Fs%2Farticle%2FListing-and-Purpose-of-Each-Log-File-Generated-by-Ivanti-Endpoint-Security-Heat-EMSS](https://forums.ivanti.com/s/article/Listing-and-Purpose-of-Each-Log-File-Generated-by-Ivanti-Endpoint-Security-Heat-EMSS?nocache=https://forums.ivanti.com/s/article/Listing-and-Purpose-of-Each-Log-File-Generated-by-Ivanti-Endpoint-Security-Heat-EMSS)  
69. Where are all the log files stored? \- Ivanti Community, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/Where-are-all-the-log-files-stored](https://forums.ivanti.com/s/article/Where-are-all-the-log-files-stored)  
70. Log files \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ld/help/en\_us/ldsd/12.0/content/004%20Administrator/Diagnostics/LogFiles.htm](https://help.ivanti.com/ld/help/en_us/ldsd/12.0/content/004%20Administrator/Diagnostics/LogFiles.htm)  
71. Obtaining various log files from Ivanti Device and Application Control (IDAC/LES/HES), 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/Obtaining-various-log-files-from-Ivanti-Device-and-Application-Control-IDAC-LES-HES](https://forums.ivanti.com/s/article/Obtaining-various-log-files-from-Ivanti-Device-and-Application-Control-IDAC-LES-HES)  
72. Where are all the log files stored? \- Ivanti Community, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/Where-are-all-the-log-files-stored?language=en\_US](https://forums.ivanti.com/s/article/Where-are-all-the-log-files-stored?language=en_US)  
73. Ivanti Connect Secure: Security Configuration Best Practices, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/KB29805?language=en\_US](https://forums.ivanti.com/s/article/KB29805?language=en_US)  
74. Log Files \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/ps/help/en\_us/vwaf/4.9/ag/vwaf\_integrated/guid-c57542fc-a11e-4c76-9aba-bcd86b11500c.html](https://help.ivanti.com/ps/help/en_us/vwaf/4.9/ag/vwaf_integrated/guid-c57542fc-a11e-4c76-9aba-bcd86b11500c.html)  
75. Risk-Based Vulnerability Management (RBVM) \- Ivanti, 4月 22, 2025にアクセス、 [https://www.ivanti.com/products/ivanti-neurons-for-rbvm](https://www.ivanti.com/products/ivanti-neurons-for-rbvm)  
76. Logging Options \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/iv/help/en\_US/isec/vNow/Topics/Logging\_Options.htm](https://help.ivanti.com/iv/help/en_US/isec/vNow/Topics/Logging_Options.htm)  
77. IT Security Solutions \- Ivanti, 4月 22, 2025にアクセス、 [https://www.ivanti.com/security](https://www.ivanti.com/security)  
78. Best Practices Guides \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/iv/help/en\_US/isec/vNow/Topics/Best-Practices-landing-page.htm](https://help.ivanti.com/iv/help/en_US/isec/vNow/Topics/Best-Practices-landing-page.htm)  
79. Security Advisory Blog \- Topic | Ivanti, 4月 22, 2025にアクセス、 [https://www.ivanti.com/blog/topics/security-advisory](https://www.ivanti.com/blog/topics/security-advisory)  
80. Chemical Security Assessment Tool (CSAT) Ivanti Notification \- CISA, 4月 22, 2025にアクセス、 [https://www.cisa.gov/chemical-security-assessment-tool-csat-ivanti-notification](https://www.cisa.gov/chemical-security-assessment-tool-csat-ivanti-notification)  
81. KB44152 \- Pulse Policy Secure: Security configuration best practices \- Ivanti Community, 4月 22, 2025にアクセス、 [https://forums.ivanti.com/s/article/KB44152](https://forums.ivanti.com/s/article/KB44152)  
82. AI Cybersecurity Best Practices: Meeting a Double-Edged Challenge \- Ivanti, 4月 22, 2025にアクセス、 [https://www.ivanti.com/blog/ai-cybersecurity-best-practices-meeting-a-double-edged-challenge](https://www.ivanti.com/blog/ai-cybersecurity-best-practices-meeting-a-double-edged-challenge)  
83. An Update on Ivanti's Ongoing Commitment to Enhanced Product Security, 4月 22, 2025にアクセス、 [https://www.ivanti.com/blog/an-update-on-ivantis-ongoing-commitment-to-enhanced-product-security](https://www.ivanti.com/blog/an-update-on-ivantis-ongoing-commitment-to-enhanced-product-security)  
84. Security Best Practices \- Ivanti, 4月 22, 2025にアクセス、 [https://help.ivanti.com/res/help/en\_us/iwc/2022/help/Content/10089.htm](https://help.ivanti.com/res/help/en_us/iwc/2022/help/Content/10089.htm)