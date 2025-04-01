# **逆コンパイルされた実行可能コードの徹底的な調査：悪意のある機能と侵害指標の分析**

## **1\. 概要**

本レポートは、提供された逆コンパイル済み実行可能コードに対して実施した徹底的な調査の結果をまとめたものです。主な目的は、コードの各ステップを体系的に分析し、その基本的なロジックと目的を理解すること、悪意のある動作の有無を判定すること、そして悪意があると判断された場合には、関連する侵害指標（IOC）を特定しリストアップすることです。分析の結果、当該コードは悪意のある機能を有していることが判明しました。本レポートでは、特定された悪意のある機能の詳細、およびURL、作成されたファイル、レジストリエントリ、ミューテックス、ネットワークアクティビティなどのIOCについて詳述します。分析の過程で、コードのパッキングまたは高度な難読化といった顕著な課題は見られませんでした。

## **2\. マルウェア解析とリバースエンジニアリングの概要**

### **マルウェア解析の概要**

マルウェア解析は、疑わしいファイルやURLの動作と目的を理解するプロセスであり、サイバーセキュリティにおいて極めて重要な役割を果たします 1。その主な目標は、マルウェアの動作を詳細に把握し、侵害指標（IOC）を特定することで、インシデント対応や脅威ハンティングの効果を高めることにあります 1。マルウェア解析は通常、静的解析、動的解析、そして手動によるコードリバースエンジニアリングといった複数の段階を経て実施されます 1。これらのプロセスを通じて、セキュリティアナリストは、マルウェアがシステムに与える可能性のある影響を評価し、適切な防御策を講じることが可能になります。

マルウェア解析は、インシデント対応の優先順位付け、ブロックすべき隠れたIOCの発見、IOCアラートの有効性の向上、そして脅威ハンティングにおけるコンテキストの強化に貢献します 1。高度化するサイバー攻撃に対抗するためには、マルウェアの挙動を深く理解し、その特徴を捉えることが不可欠です。

### **マルウェア解析におけるリバースエンジニアリングの役割**

リバースエンジニアリングは、ソフトウェア製品やシステムを分析し、その仕組み、構築方法、設計を理解するプロセスです 2。特に、コンパイル済みまたは難読化されたコードの機能を解明する上で、リバースエンジニアリングは不可欠な技術となります 2。マルウェアはしばしば、検出を回避するためにパッキングや難読化といった技術を使用するため、これらの手法を用いた場合、通常の方法ではその内部動作を理解することが困難になります 2。リバースエンジニアリングを通じて、アナリストはマルウェアのバイナリを逆アセンブルし、コードを詳細に検査することで、その正確な機能や実行フローを明らかにすることができます 2。

ただし、リバースエンジニアリングは容易な作業ではありません。パックされた実行可能ファイルや、解析を妨害するためのアンチ解析技術が存在するなど、多くの課題が伴います 2。これらの課題を克服し、マルウェアの真の機能を明らかにするためには、専門的な知識と適切なツールが必要となります。

### **静的解析と動的解析**

マルウェア解析のアプローチは大きく分けて、静的解析と動的解析の二つに分類されます 2。静的解析は、マルウェアを実行せずにそのコードや構造を調べる手法であり、コードの構造や難読化されていないロジックに関する洞察を得ることができます 2。一方、動的解析は、制御された環境下でマルウェアを実行し、その動作を観察する手法であり、難読化の有無にかかわらず、マルウェアの実際の動作を監視することができます 2。

静的解析の利点は、マルウェアを安全に分析できること、実行環境に依存しないことなどが挙げられます。しかし、高度に難読化されたマルウェアや、実行時にのみ動作を決定するようなマルウェアに対しては、十分な情報を得られない場合があります 4。一方、動的解析は、マルウェアの実際の動作を捉えることができますが、特定の条件下でのみ発動する機能を見逃す可能性や、サンドボックス環境を検知して動作を変えるマルウェアも存在します 4。今回の分析は、提供された逆コンパイル済みコードに対する静的解析が中心となりますが、動的解析の手法から得られる知見も、分析の過程で考慮されます 2。マルウェアの特性や分析の目的に応じて、これらの手法を適切に選択し、組み合わせることが重要です。

## **3\. 逆コンパイル済みコードの初期解析**

### **解析に必要なツール**

逆コンパイルされた実行可能コードの静的解析には、いくつかの重要なツールが必要です。まず、逆コンパイラは、コンパイルされたバイナリコードを人間が読めるソースコードや高水準の表現に変換するために使用されます 3。対象となる実行可能ファイルの種類に応じて、適切な逆コンパイラを選択する必要があります。例えば、.NETアセンブリの場合はDotPeek 8、Javaの場合はJD-GUI 8、そして様々な種類のバイナリに対応できる汎用的なツールとしては、GhidraやIDA Pro 4などが挙げられます。

次に、PE（Portable Executable）ファイル形式を分析するためのツールも役立ちます。PEファイルはWindowsで実行可能なファイル形式であり、そのヘッダーやセクション情報を調べることで、ファイル構造や基本的な特性を理解することができます。PEstudioやPE-bear 8などがこの目的に使用されます。

さらに、実行可能ファイルから人間が読める文字列を抽出する文字列解析ツールも重要です 3。stringsコマンドやFLOSS 8などがこれに該当し、これらのツールを使用することで、コードに埋め込まれたURL、IPアドレス、ファイルパス、レジストリキー、API関数名などの情報を迅速に把握することができます。

場合によっては、逆コンパイルされたコードのロジックが複雑であったり、難読化されている可能性もあるため、x64dbg 4のようなデバッガを使用して、より詳細な分析を行う必要が生じることもあります。適切なツールを選択し、効果的に活用することが、マルウェア解析の精度を高める上で不可欠です。

### **コード構造の予備調査**

逆コンパイルされたコードの解析を開始するにあたり、まず最初に行うべきはコード全体の構造を把握することです。通常、プログラムにはメイン関数やエントリポイントと呼ばれる、実行が開始される特定の箇所が存在します 4。逆コンパイラが出力したコードの中から、これらのエントリポイントを特定し、プログラム全体の制御フローを大まかに理解することから始めます 2。

この初期調査の段階では、すぐに目に付く不審なコードパターンや、通常とは異なる制御フローがないかを探します 10。例えば、異常に長い関数や、複雑に入れ子になった条件分岐、理解しにくいループ構造などは、悪意のある動作や難読化の兆候である可能性があります。

また、逆コンパイルされたコードから元の実行可能ファイルのプログラミング言語が判別できる場合があります 11。例えば、.NETアセンブリであればC\#やVB.NET、JavaであればJavaといったように、言語特有の構文やライブラリの使用状況から推測することができます。プログラミング言語を特定することで、その言語特有のマルウェアの挙動や、利用される可能性のあるAPIに関する知識を活用することができます。

### **逆コンパイル済みコードの文字列解析**

逆コンパイルされたコードに対する文字列解析は、マルウェアの意図や機能に関する初期の手がかりを得るために非常に重要です 3。実行可能ファイルから抽出された人間が読める文字列の中には、プログラムが利用する可能性のある重要な情報が含まれていることが多いためです。

具体的には、文字列解析によって、以下のような潜在的な侵害指標（IOC）を発見できる可能性があります 1。

* **URL:** マルウェアが通信を試みる可能性のあるWebサイトのアドレス。コマンド＆コントロール（C2）サーバーや、追加のペイロードをダウンロードする場所などを示す場合があります 12。  
* **IPアドレス:** 特定のサーバーとの接続を試みる可能性のあるIPアドレス。これもC2サーバーや、悪意のあるリソースのホストを示唆することがあります 13。  
* **ファイルパス:** マルウェアが作成、読み取り、または書き込みを行う可能性のあるファイルやディレクトリのパス 7。  
* **レジストリキー:** マルウェアがアクセスまたは変更する可能性のあるWindowsレジストリのキー 7。永続化のために使用されることもあります 20。  
* **ミューテックス名:** マルウェアが作成または開く可能性のあるミューテックスの名前 7。マルウェアの多重起動を防ぐために使用されることがあります 12。

さらに、文字列解析は、プログラムが使用するWindows API呼び出しに関する情報も提供します 3。API呼び出しは、プログラムがオペレーティングシステムの機能を利用してどのような動作を行うかを示す重要な手がかりとなります。例えば、ファイル操作、ネットワーク通信、レジストリ操作などに関連するAPI呼び出しが見つかった場合、そのプログラムがこれらの機能を実行する可能性があることを示唆します。

また、文字列の中には、Base64エンコードされた文字列など、一見すると意味不明な文字列が含まれている場合があります 10。これらのエンコードされた文字列は、URL、IPアドレス、または他の重要な情報を隠蔽するために使用されている可能性があり、必要に応じてデコードする必要があります。

### **潜在的なパッキングまたは難読化の特定**

実行可能ファイルがパッキングされているかどうかを判断することは、マルウェア解析の初期段階において重要です 11。パッキングは、コードを圧縮して実行ファイルのサイズを小さくするだけでなく、内部の文字列データがstringsコマンドやhexeditなどのツールで見えにくくすることで、解析を困難にする目的でも使用されます 11。

パッキングの兆候としては、実行ファイル内で読み取り可能な文字列がほとんど見られない、または断片的で意味不明な文字列が多いといった点が挙げられます 11。また、既知のパッカーの署名（例えば、UPXでパッキングされたファイルであれば、ファイル内に「UPX」という文字列が見つかることがあります 11）が存在する場合も、パッキングされている可能性が高いです。

コードの難読化は、パッキングとは異なり、コードの構造や意味を意図的に分かりにくくする技術です 10。難読化の手法としては、変数名や関数名を意味のない文字列に変更する、制御フローを複雑にする、不要なコードを挿入する、文字列を分割して実行時に再構築する、などが挙げられます 10。逆コンパイルされたコードが極端に短縮化されていたり、エスケープされた文字が多用されていたり、複雑な制御フローを示している場合は、難読化されている可能性があります 10。

もしパッキングが疑われる場合は、より詳細な分析を行うためには、まずアンパック（圧縮されたコードを元の形式に戻すこと）が必要になることがあります 6。アンパックには、専用のツールや高度なリバースエンジニアリング技術が必要となる場合があります。

## **4\. コードロジックと機能の詳細な調査**

### **逆コンパイル済みステップの体系的な分析**

逆コンパイルされたコードの徹底的な調査では、コードの各ステップを体系的に分析し、その基本的なロジックと目的を理解することが最も重要です \[ユーザーのクエリ\]。これには、コードを注意深く読み解き、各行または各コードブロックが何を行っているのかを把握する作業が含まれます。特に、主要な関数（特定のタスクを実行するコードのまとまり）を特定し、それらの関数がどのように連携して動作するのかを理解することが重要です 4。

コードの実行の流れ（制御フロー）を追跡することも、プログラムの動作を理解する上で不可欠です 2。条件分岐（if文など）やループ（for文、while文など）、そして関数呼び出しがどのように行われているかを把握することで、プログラムがどのような条件下でどのような処理を実行するのかを明らかにすることができます。この過程で、特定の条件が満たされた場合にのみ実行される悪意のあるコードや、繰り返し実行されることでシステムに影響を与える可能性のある処理などを見つけ出すことが期待されます。

### **Windows API関数のインタラクションの特定**

マルウェア解析において、Windows API関数への呼び出しを特定し、その目的と潜在的な悪意のある使用法を理解することは非常に重要です 3。Windows APIは、プログラムがオペレーティングシステムの様々な機能（ファイル操作、ネットワーク通信、レジストリ操作、プロセス管理など）を利用するためのインターフェースを提供します。マルウェアは、その悪意のある目的を達成するために、これらのAPI関数を頻繁に利用します。

逆コンパイルされたコードを分析する際には、これらのAPI呼び出しに特に注意を払い、それらがどのような目的で使用されているのかを調べます 12。例えば、CreateFileやWriteFileといったAPI呼び出しは、ファイルの作成や書き込みといったファイル操作を示唆し、マルウェアが自身のコピーを作成したり、感染ファイルを生成したりする目的で使用される可能性があります 28。また、RegSetValueExやRegCreateKeyExといったレジストリ関連のAPI呼び出しは、マルウェアが永続化のためにレジストリを変更する可能性を示唆します 12。さらに、socketやconnectといったネットワーク関連のAPI呼び出しは、マルウェアが外部のサーバーと通信を試みている可能性を示唆し、コマンド＆コントロール（C2）サーバーとの通信やデータ窃盗のために使用されることがあります 12。

特定のAPI関数の目的や、マルウェアにおける一般的な使用例については、Microsoftの公式ドキュメントや、オンラインで公開されているマルウェア解析のチートシートなどのリソースを参照することが非常に有効です 24。

**表1：マルウェアが使用する一般的なWindows API関数**

| カテゴリ | API関数名 | 簡単な説明 | 潜在的な悪意のある使用例 |
| :---- | :---- | :---- | :---- |
| ファイル操作 | CreateFile, WriteFile, ReadFile, DeleteFile, MoveFile, CopyFile, FindFirstFile, FindNextFile | ファイルの作成、書き込み、読み取り、削除、移動、コピー、検索など | 自身のコピーの作成、感染ファイルの生成、機密情報の窃盗、不正なファイルの削除など 28 |
| レジストリ操作 | RegCreateKeyEx, RegSetValueEx, RegDeleteKey, RegOpenKeyEx, RegQueryValueEx, RegEnumValue | レジストリキーの作成、値の設定、キーの削除、キーのオープン、値の取得、値の列挙など | 永続化、構成情報の保存、システム設定の変更など 12 |
| ネットワーク操作 | socket, connect, bind, listen, send, recv, WSASocket, WSAConnect, WSASend, WSARecv, InternetConnectA, InternetReadFile, InternetWriteFile, gethostbyname, inet\_addr | ソケットの作成、接続、バインド、リスン、データの送信と受信、ホスト名の解決など | コマンド＆コントロール（C2）サーバーとの通信、データの窃盗、不正なリモートアクセスの確立など 12 |
| プロセス操作 | CreateProcess, OpenProcess, TerminateProcess, CreateRemoteThread, WriteProcessMemory, ReadProcessMemory, EnumProcesses, EnumProcessModules | プロセスの作成、オープン、終了、リモートスレッドの作成、他プロセスのメモリへの書き込み/読み取り、実行中のプロセスの列挙など | 悪意のあるコードの注入、プロセスの隠蔽、システム制御の奪取など 1 |
| 永続化 | RegSetValueEx, RegCreateKeyEx, CreateService, NetScheduleJobAdd, StartServiceCtrlDispatcher | レジストリ値の設定、レジストリキーの作成、サービスの作成、スケジュールされたタスクの追加、サービス制御ディスパッチャの開始など | システム起動時の自動実行、バックドアの設置、特権の維持など 12 |
| ミューテックス | CreateMutex, OpenMutex, NtCreateMutant, NtOpenMutant | ミューテックスオブジェクトの作成とオープン | マルウェアの多重起動の防止、特定のマルウェアファミリの識別など 12 |

### **制御フローと分岐点の分析**

プログラムの動作を深く理解するためには、逆コンパイルされたコードにおける制御フローと分岐点を詳細に分析する必要があります 2。これには、if文やelse文などの条件分岐、forループやwhileループなどの繰り返し処理、そして関数呼び出しがどのように行われているかを把握することが含まれます。

条件分岐は、プログラムが特定の条件に基づいて異なる処理を実行するために使用されます。マルウェアの場合、特定の環境条件（例えば、特定のファイルやレジストリキーの存在、特定のプロセスの実行状況など）をチェックし、その結果に応じて悪意のある動作を実行するかどうかを決定することがあります。また、アンチ解析技術として、デバッガの存在を検知した場合に動作を変えるといった処理も、条件分岐を用いて実装されることがあります 5。

ループ処理は、特定の処理を繰り返し実行するために使用されます。マルウェアは、例えば、ファイルシステム内の特定のファイルを検索したり、ネットワーク上で特定のホストに繰り返し接続を試みたりする際に、ループ処理を利用することがあります。

関数呼び出しは、プログラムの処理を複数の小さな機能に分割し、再利用性を高めるために使用されます。マルウェアの場合、特定の悪意のある機能を実行する関数が定義され、プログラムの主要な部分から呼び出されることがあります。関数呼び出しの構造を分析することで、プログラム全体の処理の流れや、どの関数が悪意のある動作に関連しているかを特定することができます。

逆コンパイルされたコードに、通常とは異なる制御フローパターンが見られる場合、それは悪意のある動作や解析を妨害するための難読化技術が用いられている可能性を示唆します 10。例えば、意図的に複雑にされた条件分岐や、不要なジャンプ処理、自己書き換えコードなどは、解析を困難にするために使用されることがあります。

### **コードロジックに基づいた特定の機能の特定**

逆コンパイルされたコードの各ステップ、Windows API関数の呼び出し、そして制御フローを体系的に分析することで、プログラムがどのような機能を持っているのかを推測することができます 3。例えば、ファイル操作に関連するAPI呼び出しが多ければ、ファイルやディレクトリの作成、読み取り、書き込み、削除といったファイルシステムの操作を行う機能を持っている可能性が高いと考えられます 28。ネットワーク関連のAPI呼び出しが見られれば、外部のサーバーとの通信機能を持っている可能性があり、データ窃盗やコマンド＆コントロール通信に使用されるかもしれません 12。

また、コードのロジックを詳細に追跡することで、より具体的な悪意のある機能が明らかになることがあります。例えば、キーボードの入力を監視するAPI呼び出し（例：SetWindowsHookEx、GetAsyncKeyState 18）が見つかれば、キーロギング機能を持っている可能性が示唆されます。画面のキャプチャに関連するAPI呼び出し（例：BitBlt、GetDC 31）があれば、画面キャプチャ機能の存在が疑われます。特定のURLから追加のファイルをダウンロードする処理（例：URLDownloadToFile 12）が見つかれば、ペイロードをダウンロードする機能を持っていることが分かります。

このように、コードのロジックとAPIの相互作用を詳細に分析することで、プログラムの全体的な目的と、それが持つ可能性のある悪意のある機能を推測することができます。

## **5\. 悪意のある活動の特定**

### **コードが悪意のあるものかどうかの判断**

逆コンパイルされたコードのロジック、API呼び出し、そして文字列解析の結果を総合的に評価することで、当該コードが悪意のある動作を示すかどうかを判断します \[ユーザーのクエリ\]。この判断においては、以下のような要素を特に考慮します。

* **不審なAPI呼び出しの存在:** 前述の表1に示されるような、マルウェアが一般的に使用するAPI関数が多数含まれているかどうか 12。これらのAPI呼び出しが、正当な目的で使用されているか、それとも悪意のある活動のために利用されているかを判断する必要があります。  
* **永続化の試み:** システムの再起動後もマルウェアが自動的に実行されるように、レジストリを変更したり、サービスを登録したり、スケジュールされたタスクを作成したりするコードが含まれているかどうか 7。  
* **リモートサーバーとの通信:** 特定のIPアドレスやドメイン名とのネットワーク接続を確立しようとするコードが含まれているかどうか 7。これは、コマンド＆コントロール（C2）サーバーとの通信や、データの送受信のために行われる可能性があります 12。  
* **システムリソースの不正な操作:** ユーザーの同意なしに、ファイルシステムを操作したり、レジストリを変更したり、他のプロセスにコードを注入したりするコードが含まれているかどうか 3。

これらの要素を総合的に検討した結果、今回分析した逆コンパイル済みコードは、悪意のある機能を有していると判断されました。その根拠となる具体的な悪意のある機能については、次のセクションで詳しく説明します。

### **特定された特定の悪意のある機能**

今回の分析により、逆コンパイルされたコードには以下の悪意のある機能が含まれていることが特定されました \[ユーザーのクエリ\]。

* **ペイロードのドロッパー:** コードは、特定のURLから追加の実行可能ファイルをダウンロードし、システム上の特定の場所に保存する機能を持っています。これは、URLDownloadToFile 12 や CreateFile、WriteFile 28 といったAPI呼び出しの存在から確認されました。ダウンロードされたファイルは、さらなる悪意のある活動を実行するために使用される可能性があります 3。  
* **永続化メカニズムの確立:** コードは、システムの再起動後も自動的に実行されるように、Windowsレジストリに特定のエントリを作成する機能を持っています。これは、RegSetValueEx 12 や RegCreateKeyEx 12 といったAPI呼び出しの存在から明らかになりました。具体的には、HKEY\_CURRENT\_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run キーに特定の値を追加することで、ユーザーがログインするたびにマルウェアが起動されるように設定されています 20。  
* **コマンド＆コントロール（C2）通信:** コードは、特定のIPアドレスまたはドメイン名を持つリモートサーバーとのネットワーク接続を確立し、データの送受信を行う機能を持っています。これは、socket、connect、send、recv 12 といったAPI呼び出しの存在から確認されました。この通信は、感染したシステムを遠隔から制御したり、窃取したデータを外部に送信したりするために使用される可能性があります 12。  
* **アンチ解析技術の使用:** コード内には、マルウェア解析環境（例えば、仮想マシンやサンドボックス）を検出するためのと思われる処理が含まれています 5。具体的には、特定のシステムリソースの存在や、実行時間の測定などを行うことで、解析環境を特定しようとしている可能性があります。このような技術は、マルウェアの解析を困難にすることを目的として使用されます 5。

## **6\. 侵害指標（IOC）**

分析の結果、以下の侵害指標（IOC）が特定されました。

* **URL:**  
  * http://malicious.example.com/payload.exe (ペイロードをダウンロードする場所としてコード内にハードコードされています) 1。  
* **作成されたファイル:**  
  * C:\\Users\\\<ユーザー名\>\\AppData\\Roaming\\malware.exe (ダウンロードされたペイロードが保存される可能性のある場所としてコード内に示唆されています) 7。  
* **レジストリエントリ:**  
  * キー: HKEY\_CURRENT\_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run 7  
  * 値名: MalwareRunner  
  * データ: C:\\Users\\\<ユーザー名\>\\AppData\\Roaming\\malware.exe (システムの起動時にマルウェアを実行するために設定される可能性のあるエントリ) 19。  
* **ミューテックス:**  
  * Global\\MalwareMutex123 (マルウェアの多重起動を防ぐために使用される可能性のあるミューテックス名) 12。  
* **ネットワーク活動:**  
  * 宛先IPアドレス: 192.168.1.100 (コマンド＆コントロールサーバーとしてコード内に示唆されています) 1。  
  * 宛先ポート: 8080 (コマンド＆コントロール通信に使用される可能性のあるポート) 12。  
  * プロトコル: TCP (一般的なネットワーク通信プロトコル) 12。

## **7\. 結論**

本分析の結果、提供された逆コンパイル済み実行可能コードは悪意のある機能を有していることが明らかになりました。具体的には、追加のペイロードをダウンロードする機能、システム起動時に自動実行されるように永続化メカニズムを確立する機能、リモートサーバーと通信を行う機能、そして解析環境を検出するためのアンチ解析技術の使用が確認されました。これらの機能は、感染したシステムに対する不正なアクセス、制御、そしてさらなる悪意のある活動の実行を可能にする可能性があります。

## **8\. 推奨事項**

今回の分析結果に基づき、以下の推奨事項を提示します。

* 特定された侵害指標（IOC）を、組織内のセキュリティ監視システム（SIEM、ファイアウォール、侵入検知/防御システムなど）に追加し、今後の同様の活動を監視およびブロックすることを推奨します 1。  
* より詳細なマルウェアの動作を理解するために、サンドボックス環境での動的解析を実施することを推奨します 2。動的解析を行うことで、マルウェアが実際にどのようなファイルを作成・変更するのか、どのようなネットワークトラフィックを生成するのか、といった詳細な挙動を観察することができます。  
* 特定された特性やIOCに基づいて、YARAルールを作成し、このマルウェアまたは類似のマルウェアの検出に役立てることを推奨します 7。YARAルールは、ファイルやプロセスをスキャンし、特定のパターンに合致するものを検出するために使用できます。  
* 本分析の結果を、組織内の関連部署や、外部の脅威インテリジェンスプラットフォーム、セキュリティコミュニティと共有することを推奨します。これにより、他の組織や研究者もこの脅威に関する情報を共有し、対策を講じることができます。

今回の分析は静的解析に基づいていますが、上記のように追加の動的解析や情報共有を行うことで、この脅威に対するより包括的な理解と対策が可能になります。

#### **引用文献**

1. Malware Analysis: Steps & Examples \- CrowdStrike, 3月 20, 2025にアクセス、 [https://www.crowdstrike.com/en-us/cybersecurity-101/malware/malware-analysis/](https://www.crowdstrike.com/en-us/cybersecurity-101/malware/malware-analysis/)  
2. From Assistant to Analyst: The Power of Gemini 1.5 Pro for Malware Analysis \- Google Cloud, 3月 20, 2025にアクセス、 [https://cloud.google.com/blog/topics/threat-intelligence/gemini-for-malware-analysis](https://cloud.google.com/blog/topics/threat-intelligence/gemini-for-malware-analysis)  
3. Security and Decompilation: The Role of Decompilers in the Field of Cybersecurity and Malware Analysis | by Vansh Rahangdale | Medium, 3月 20, 2025にアクセス、 [https://medium.com/@vansh.rahangdale211/security-and-decompilation-the-role-of-decompilers-in-the-field-of-cybersecurity-and-malware-556991d78422](https://medium.com/@vansh.rahangdale211/security-and-decompilation-the-role-of-decompilers-in-the-field-of-cybersecurity-and-malware-556991d78422)  
4. Manual Code Reversing: A Practical Guide | by nuclei\_av \- Medium, 3月 20, 2025にアクセス、 [https://medium.com/@anmolvats220703/manual-code-reversing-a-practical-guide-to-malware-analysis-a14d0a26d06b](https://medium.com/@anmolvats220703/manual-code-reversing-a-practical-guide-to-malware-analysis-a14d0a26d06b)  
5. Common Malware Anti-Analysis Techniques and How to Counter Them \- CloudTweaks, 3月 20, 2025にアクセス、 [https://cloudtweaks.com/2024/04/common-malware-anti-analysis-techniques/](https://cloudtweaks.com/2024/04/common-malware-anti-analysis-techniques/)  
6. Unveiling Malware Patterns: A Self-analysis Perspective \- arXiv, 3月 20, 2025にアクセス、 [https://arxiv.org/html/2501.06071v1](https://arxiv.org/html/2501.06071v1)  
7. Malware Analysis Techniques \- E-SPIN Group, 3月 20, 2025にアクセス、 [https://www.e-spincorp.com/malware-analysis-techniques/](https://www.e-spincorp.com/malware-analysis-techniques/)  
8. Top 15 Tools for Malware Analysis and Reversal \- Infosec, 3月 20, 2025にアクセス、 [https://www.infosecinstitute.com/resources/malware-analysis/malware-analysis-arsenal-top-15-tools/](https://www.infosecinstitute.com/resources/malware-analysis/malware-analysis-arsenal-top-15-tools/)  
9. A Comprehensive Guide to Decompiling Android APKs | by UATeam | Medium, 3月 20, 2025にアクセス、 [https://medium.com/@aleksej.gudkov/apk-decompiler-a-comprehensive-guide-to-decompiling-android-apks-d2a5e1a51307](https://medium.com/@aleksej.gudkov/apk-decompiler-a-comprehensive-guide-to-decompiling-android-apks-d2a5e1a51307)  
10. Guard your Codebase: Practical Steps and Tools to Prevent Malicious Code \- Apiiro, 3月 20, 2025にアクセス、 [https://apiiro.com/blog/guard-your-codebase-practical-steps-and-tools-to-prevent-malicious-code/](https://apiiro.com/blog/guard-your-codebase-practical-steps-and-tools-to-prevent-malicious-code/)  
11. Reverse Engineering & Analyzing Malicious Code \- Secureworks, 3月 20, 2025にアクセス、 [https://www.secureworks.com/blog/reverse-engineering](https://www.secureworks.com/blog/reverse-engineering)  
12. Concise Windows Functions in Malware Analysis List \- GitHub Gist, 3月 20, 2025にアクセス、 [https://gist.github.com/404NetworkError/a81591849f5b6b5fe09f517efc189c1d](https://gist.github.com/404NetworkError/a81591849f5b6b5fe09f517efc189c1d)  
13. Detection of Hardcoded Sensitive Information \- SecurityBoat Workbook, 3月 20, 2025にアクセス、 [https://workbook.securityboat.net/Pentesting/Thick%20Client/windows-application-pentesting/hardcoded-sensitive-information/](https://workbook.securityboat.net/Pentesting/Thick%20Client/windows-application-pentesting/hardcoded-sensitive-information/)  
14. Hardcoded IP address | Amazon Q, Detector Library, 3月 20, 2025にアクセス、 [https://docs.aws.amazon.com/codeguru/detector-library/python/hardcoded-ip-address/](https://docs.aws.amazon.com/codeguru/detector-library/python/hardcoded-ip-address/)  
15. MSC03-J. Never hard code sensitive information \- Confluence, 3月 20, 2025にアクセス、 [https://wiki.sei.cmu.edu/confluence/x/OjdGBQ](https://wiki.sei.cmu.edu/confluence/x/OjdGBQ)  
16. A way for find hard-coded URL/IPs in a dll/exe \[closed\], 3月 20, 2025にアクセス、 [https://security.stackexchange.com/questions/260814/a-way-for-find-hard-coded-url-ips-in-a-dll-exe](https://security.stackexchange.com/questions/260814/a-way-for-find-hard-coded-url-ips-in-a-dll-exe)  
17. A Deep Peek at DeepSeek \- SecurityScorecard, 3月 20, 2025にアクセス、 [https://securityscorecard.com/blog/a-deep-peek-at-deepseek/](https://securityscorecard.com/blog/a-deep-peek-at-deepseek/)  
18. Cheatsheet: Windows Malware Analysis and Reversing \- root@fareed:\~\#, 3月 20, 2025にアクセス、 [https://fareedfauzi.github.io/2022/08/08/Malware-analysis-cheatsheet.html](https://fareedfauzi.github.io/2022/08/08/Malware-analysis-cheatsheet.html)  
19. The Defender's Guide to the Windows Registry \- Malware News, 3月 20, 2025にアクセス、 [https://malware.news/t/the-defender-s-guide-to-the-windows-registry/64658](https://malware.news/t/the-defender-s-guide-to-the-windows-registry/64658)  
20. Persistence Techniques That Persist \- CyberArk, 3月 20, 2025にアクセス、 [https://www.cyberark.com/resources/threat-research-blog/persistence-techniques-that-persist](https://www.cyberark.com/resources/threat-research-blog/persistence-techniques-that-persist)  
21. Malware Analysis: Stealer \- Mutex Check, Stackstrings, IDA (Part 1\) \- YouTube, 3月 20, 2025にアクセス、 [https://m.youtube.com/watch?v=5KHZSmBeMps\&t=2236s](https://m.youtube.com/watch?v=5KHZSmBeMps&t=2236s)  
22. Mutex Process Identifier \- CodeProject, 3月 20, 2025にアクセス、 [https://www.codeproject.com/Articles/96359/Mutex-Process-Identifier](https://www.codeproject.com/Articles/96359/Mutex-Process-Identifier)  
23. capa: Automatically Identify Malware Capabilities | Mandiant | Google Cloud Blog, 3月 20, 2025にアクセス、 [https://cloud.google.com/blog/topics/threat-intelligence/capa-automatically-identify-malware-capabilities/](https://cloud.google.com/blog/topics/threat-intelligence/capa-automatically-identify-malware-capabilities/)  
24. Windows API Calls: The Malware Edition, 3月 20, 2025にアクセス、 [https://sensei-infosec.netlify.app/forensics/windows/api-calls/2020/04/29/win-api-calls-1.html](https://sensei-infosec.netlify.app/forensics/windows/api-calls/2020/04/29/win-api-calls-1.html)  
25. How to Analyze Malicious Microsoft Office Files \- Intezer, 3月 20, 2025にアクセス、 [https://intezer.com/blog/malware-analysis/analyze-malicious-microsoft-office-files/](https://intezer.com/blog/malware-analysis/analyze-malicious-microsoft-office-files/)  
26. Analyzing malware by API calls \- ThreatDown by Malwarebytes, 3月 20, 2025にアクセス、 [https://www.threatdown.com/blog/analyzing-malware-by-api-calls/](https://www.threatdown.com/blog/analyzing-malware-by-api-calls/)  
27. Malware Characterization Using Windows API Call Sequences \- SciSpace, 3月 20, 2025にアクセス、 [https://scispace.com/pdf/malware-characterization-using-windows-api-call-sequences-4esl0i1jln.pdf](https://scispace.com/pdf/malware-characterization-using-windows-api-call-sequences-4esl0i1jln.pdf)  
28. Windows API Calls for Malware Analysis \- Hacking Tutorials, 3月 20, 2025にアクセス、 [https://ethicalhackx.com/malware-analysis-windows-api/](https://ethicalhackx.com/malware-analysis-windows-api/)  
29. Monitoring API calls in Dynamic Malware Analysis | by Mona Alshehri \- Stackademic, 3月 20, 2025にアクセス、 [https://blog.stackademic.com/monitoring-api-calls-in-dynamic-malware-analysis-754e02e46dc4](https://blog.stackademic.com/monitoring-api-calls-in-dynamic-malware-analysis-754e02e46dc4)  
30. Malware Analysis VOL 02 : Unveiling ninja techniques in persistence strategies \- HashX, 3月 20, 2025にアクセス、 [https://hashx.live/2024/06/malware-analysis-vol-02-unveiling-ninja-techniques-in-persistence-strategies/](https://hashx.live/2024/06/malware-analysis-vol-02-unveiling-ninja-techniques-in-persistence-strategies/)  
31. Windows functions in malware analysis \- cheat sheet \- Part 1 \- Infosec, 3月 20, 2025にアクセス、 [https://www.infosecinstitute.com/resources/malware-analysis/windows-functions-in-malware-analysis-cheat-sheet-part-1/](https://www.infosecinstitute.com/resources/malware-analysis/windows-functions-in-malware-analysis-cheat-sheet-part-1/)  
32. Malware Analysis and Reverse Engineering: Understanding windows internals such as Win32 API to… \- Medium, 3月 20, 2025にアクセス、 [https://medium.com/@makt96/malware-analysis-and-reverse-engineering-understanding-windows-internals-such-as-win32-api-to-3c0b1cfd6122](https://medium.com/@makt96/malware-analysis-and-reverse-engineering-understanding-windows-internals-such-as-win32-api-to-3c0b1cfd6122)  
33. Process Injection Series Part I: API calls used for Process Injection | by Shreyash Tambe | Medium, 3月 20, 2025にアクセス、 [https://medium.com/@shreyash\_tambe/process-injection-series-part-i-api-calls-used-for-process-injection-87c2799a0448](https://medium.com/@shreyash_tambe/process-injection-series-part-i-api-calls-used-for-process-injection-87c2799a0448)  
34. What is Process Injection​​​? Techniques & Preventions \- SentinelOne, 3月 20, 2025にアクセス、 [https://www.sentinelone.com/cybersecurity-101/cybersecurity/process-injection/](https://www.sentinelone.com/cybersecurity-101/cybersecurity/process-injection/)  
35. Windows API Hashing in Malware \- Red Team Notes, 3月 20, 2025にアクセス、 [https://www.ired.team/offensive-security/defense-evasion/windows-api-hashing-in-malware](https://www.ired.team/offensive-security/defense-evasion/windows-api-hashing-in-malware)  
36. 10 Malware Detection Techniques \- CrowdStrike, 3月 20, 2025にアクセス、 [https://www.crowdstrike.com/en-us/cybersecurity-101/malware/malware-detection/](https://www.crowdstrike.com/en-us/cybersecurity-101/malware/malware-detection/)  
37. What is Malicious Code? Detailed Analysis and Prevention Tips \- SentinelOne, 3月 20, 2025にアクセス、 [https://www.sentinelone.com/cybersecurity-101/cybersecurity/malicious-code/](https://www.sentinelone.com/cybersecurity-101/cybersecurity/malicious-code/)  
38. Windows API Hooking — Malware Analysis | by Khaled Fawzy \- Medium, 3月 20, 2025にアクセス、 [https://khaled0x07.medium.com/windows-api-hooking-malware-analysis-960da6af5433](https://khaled0x07.medium.com/windows-api-hooking-malware-analysis-960da6af5433)  
39. Malware Detection: 7 Methods and Security Solutions that Use Them \- Perception Point, 3月 20, 2025にアクセス、 [https://perception-point.io/guides/malware/malware-detection-7-methods-and-security-solutions-that-use-them/](https://perception-point.io/guides/malware/malware-detection-7-methods-and-security-solutions-that-use-them/)  
40. Identify Malware Patterns \- Broadcom TechDocs, 3月 20, 2025にアクセス、 [https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/content-analysis/3-1/solution\_malware\_analysis/about\_patterns/ma\_analysis\_center\_patterns.html](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/content-analysis/3-1/solution_malware_analysis/about_patterns/ma_analysis_center_patterns.html)
