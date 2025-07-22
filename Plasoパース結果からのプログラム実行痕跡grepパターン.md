## **Plasoパース結果からのプログラム実行痕跡grepパターン**

plasoのパース結果（通常はCSV形式）から、Windowsのプログラム実行痕跡をgrepコマンドで検索するための主要なパターンを以下にまとめています。これらのコマンドは、plaso\_output.csvというファイル名で出力が保存されていることを前提としています。すべてのgrepコマンドに-aオプション（バイナリファイルをテキストとして扱う）を追加しました。

### **全てのgrepパターン**

\# Prefetchファイル (.pf)  
grep \-a "data\_type: windows:prefetch" plaso\_output.csv | grep \-i "program\_name.exe"  
grep \-a "data\_type: windows:prefetch" plaso\_output.csv | grep "run\_count: \[0-9\]"

\# ShimCache (AppCompatCache)  
grep \-a "data\_type: windows:registry:shimcache" plaso\_output.csv | grep \-i "program\_name.exe"  
grep \-a "data\_type: windows:registry:shimcache" plaso\_output.csv

\# Amcache.hve  
grep \-a "data\_type: windows:amcache" plaso\_output.csv | grep \-i "program\_name.exe"  
grep \-a "data\_type: windows:amcache" plaso\_output.csv | grep "sha1: \<SHA1ハッシュ値\>"

\# UserAssist  
grep \-a "data\_type: windows:registry:userassist" plaso\_output.csv | grep \-i "program\_name.exe"  
grep \-a "data\_type: windows:registry:userassist" plaso\_output.csv

\# イベントログ (Event Logs) \- セキュリティログ (Event ID 4688\)  
grep \-a "data\_type: windows:eventlog:security" plaso\_output.csv | grep "Event ID: 4688"  
grep \-a "data\_type: windows:eventlog:security" plaso\_output.csv | grep "Event ID: 4688" | grep \-i "New Process Name: .\*program\_name.exe"

\# イベントログ (Event Logs) \- Sysmon (Event ID 1\)  
grep \-a "data\_type: windows:eventlog:microsoft-windows-sysmon/operational" plaso\_output.csv | grep "Event ID: 1"  
grep \-a "data\_type: windows:eventlog:microsoft-windows-sysmon/operational" plaso\_output.csv | grep "Event ID: 1" | grep \-i "Image: .\*program\_name.exe"

\# レジストリのRunキー (Run Keys)  
grep \-a "data\_type: windows:registry:run" plaso\_output.csv  
grep \-a "data\_type: windows:registry:runonce" plaso\_output.csv  
grep \-a "data\_type: windows:registry:run" plaso\_output.csv | grep \-i "program\_name.exe"

\# タスクスケジューラ (Task Scheduler)  
grep \-a "data\_type: windows:task\_scheduler:task" plaso\_output.csv  
grep \-a "data\_type: windows:task\_scheduler:task" plaso\_output.csv | grep \-i "program\_name.exe"

\# ジャンプリスト (Jump Lists)  
grep \-a "data\_type: windows:jumplist:entry" plaso\_output.csv | grep \-i "program\_name.exe"  
grep \-a "data\_type: windows:jumplist:entry" plaso\_output.csv | grep \-i "document.docx"

\# 最近使ったファイル (RecentFiles/LNKファイル)  
grep \-a "data\_type: windows:lnk" plaso\_output.csv | grep \-i "program\_name.exe"  
grep \-a "data\_type: windows:lnk" plaso\_output.csv | grep \-i "document.pdf"

\# Windows Defender / その他のアンチウイルスログ  
grep \-a "data\_type: windows:eventlog" plaso\_output.csv | grep \-i "Windows Defender"  
grep \-a "data\_type: windows:eventlog" plaso\_output.csv | grep \-i "malware"

\# 全ての主要なdata\_typeをまとめて検索 (egrep \-E または grep \-E を使用)  
\# このコマンドは、指定されたdata\_typeのいずれかを含む行を検索します。  
\# 特定のプログラム名やキーワードでさらに絞り込む場合は、パイプでgrepを連結してください。  
grep \-a \-E "data\_type: (windows:prefetch|windows:registry:shimcache|windows:amcache|windows:registry:userassist|windows:eventlog:security|windows:eventlog:microsoft-windows-sysmon/operational|windows:registry:run|windows:registry:runonce|windows:task\_scheduler:task|windows:jumplist:entry|windows:lnk)" plaso\_output.csv

### **grep使用時のヒント**

grep使用時のヒント: \-a (バイナリをテキストとして扱う), \-i (大文字小文字を区別しない), \-A N/-B N/-C N (前後の行を表示), \--color=auto (色付け), | (パイプで連結), 正規表現 (複雑なパターンマッチング) を活用します。