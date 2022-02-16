# linuxメモリのfreeとmeminfoの関係を図解し利用率の計算方法を説明してみる #  
  
# はじめに  
linuxサーバを利用する上で何時も頭を悩ますものの一つが、メモリ利用状況の評価(メモリ利用率)ではないでしょうか。    
私も悩みます。そこでRHELベースですが、メモリ利用容量/メモリ利用率の評価をどう考えるかということを整理しました。    
詳細は後述しますが、例えばRHEL7のfreeコマンドとmeminfoの関係を整理するとこんな感じになります。  
  
  
  
ちなみにlinuxのメモリ解説＆meminfoをどのように見ればいいかは、e.nakaiさんのページが参考になります。(私のメモリ周辺のバイブル的ページです^^;)  
  
  
# linuxのメモリ利用容量(空き容量)の考え方  
  
何を持って、「メモリが空いている」とするのか？？  
単純に考えると、何も利用していない(割り当てていない）領域を空き容量とすればいいはずです。  
しかしlinuxをはじめとする現在のOSでは、「使えるメモリはとことん使ってしまえ！！」という設計思想なため、空きメモリは積極的にファイルキャッシュ用途に利用してしまいます。  
その結果、「未割り当て領域を空き領域」と定義すると、メモリ利用率が常に100%となり「意味がねぇなぁ」という感じになってしまいます。  
  
そこで改めて「メモリが空いている状態」を考える必要があります。そこで着目するのが、メモリ解放のカーネルの挙動です。  
カーネルはプロセスやドライバから新規メモリ(ページ)の確保要求があった時、未使用領域(freeコマンドやmeminfoで言うところのfree領域)から新規割り当てを行います。  
未使用領域が無い場合、既存のメモリを解放して空きを作り、新なメモリ割当を行います。この「既存メモリの解放処理」は2通りのやり方があり、一つはスワップアウトして空きを作る方法、もう一つはディスク同期してメモリ解放する方法になります。  
メモリ解放の方式を整理すると、以下の表のようになります。  
  
  
※1 無名ページ＆ファイルページについては、記事文末を参照。  
※2 プロセスのテキスト領域とは、プロセスの実行イメージ(プログラム)を格納するメモリ領域です。  
※3 もちろん、プロセス終了やプログラム中のfree()など、本当に不要になって開放するパターンもありますが、ここでは割愛します。  
  
ここでサーバ用途の場合のメモリ設計を考えると、「可能な限りスワップを発生させないようにメモリサイジングする」ことが重要になります。  
特に(バッチ処理ではなく)オンライン系のサーバ(web,AP,DBサーバとか)の場合、スワップイン処理にかかるオーバーヘッドによるクライアントへのレスポンス遅延が一番気になるためです。  
例えば通常1秒以内でレスポンスが帰っている処理が数秒や最悪十数秒に伸びることで、利用者のユーザビリティが落ち「使えねーシステム」の烙印を押されることになることになるためです。  
  
ということで、メモリ空き容量を評価する上は、「スワップが発生しない程度」の空きが保たれていることが重要というとになり、結果として「未割当て領域＋ディスク同期による解放領域」を空きと見なすという考えになります。  
(説明がちょっと強引ですが。)  
  
ここまでの話を、式にまとめると以下のような形になります。  
  
メモリの空き容量 = 未割り当て容量(free) + ディスク同期にて解放可能な容量(ファイルページ≒バッファ＆キャッシュ)  
メモリ利用容量 = メモリ合計容量 - メモリ空き容量  
           = メモリから解放不可な容量 + 解放時にスワップアウトしてしまう容量  
  
# linuxのメモリ利用容量/空き容量の計算方法  
  
と前置きが長かったですが、ここからがlinuxの悩ましい所。  
  
なぜなら、linuxでメモリ容量を確認するfreeコマンドで取得できるバッファ＆キャッシュ領域には、スワップする領域も含まれており、単純に「メモリ利用容量＝メモリ全体 - free - Buffer&Cache」としてしまうと、メモリ利用容量を過小評価してしまうためです。  
最近のlinuxカーネルはこの辺を考慮したメモリ情報を表示してくれるのですが(RHEL7とか。詳細は後述)、昔の特にRHEL5以前は真面目にやろうとすると結構しんどかったです。  
(カーネルのメモリ管理を理解していなかったという要素も大きいですが。。。)  
  
以下に最新のRHEL7から順に、代々のバージョンでどのようにメモリ利用容量/利用率を計算していたかを説明します。こうやって整理すると、「昔は苦労したなぁ〜」というのがよくわかりますね。^^;  
## RHEL7  
  
最初にRHEL7ですが、RHEL7からはメモリ利用率の計算が劇的に楽になりました。  
というのも、上でぐちゃぐちゃ説明したことを、freeコマンドの「available」というパラメータで表示してくれるからです。(厳密には/proc/meminfoに"MemAvailable:"という利用可能なメモリ容量の目安を表示してくれるようになったのですが)  
  
なので、わざわざmeminfoをみなくても、freeコマンドの結果をちょっと加工すれば利用率をだせるようになりました。  
## 【freeコマンドとmeminfoの図解】  
  
TODO  
  
  
## 【計算方法】  
freeコマンドの結果をawkで処理させています。  
  
#!/bin/bash  
export LANG=C, LC_ALL=C  
  
free | awk '  
    BEGIN{  
        total=0;used=0;available=0;rate=0;  
    }  
  
    /^Mem:/{  
        total=$2;  
        available=$7;  
    }  
  
    END{  
        used=total-available;  
        rate=100*used/total;  
        printf("used%,total(KB),used(KB),available(KB)\n");  
        printf("%.1f,%d,%d,%d\n",rate,total,used,available);  
    }';  
  
freeコマンド表示例  
#free  
              total        used        free      shared  buff/cache   available  
Mem:        1884812      106076     1615644        8632      163092     1624364  
Swap:       2097148           0     2097148  
  
/proc/meminfo表示例  
#cat /proc/meminfo  
MemTotal:        1884812 kB  
MemFree:         1615784 kB  
MemAvailable:    1624504 kB  
Buffers:             880 kB  
Cached:           120396 kB  
SwapCached:            0 kB  
Active:            84932 kB  
Inactive:          95568 kB  
Active(anon):      59524 kB  
Inactive(anon):     8332 kB  
Active(file):      25408 kB  
Inactive(file):    87236 kB  
Unevictable:           0 kB  
Mlocked:               0 kB  
SwapTotal:       2097148 kB  
SwapFree:        2097148 kB  
Dirty:                 0 kB  
Writeback:             0 kB  
AnonPages:         59244 kB  
Mapped:            20852 kB  
Shmem:              8632 kB  
Slab:              41816 kB  
SReclaimable:      17396 kB  
SUnreclaim:        24420 kB  
KernelStack:        1680 kB  
PageTables:         3688 kB  
NFS_Unstable:          0 kB  
Bounce:                0 kB  
WritebackTmp:          0 kB  
CommitLimit:     3039552 kB  
Committed_AS:     268452 kB  
VmallocTotal:   34359738367 kB  
VmallocUsed:      156764 kB  
VmallocChunk:   34359575144 kB  
HardwareCorrupted:     0 kB  
AnonHugePages:      4096 kB  
HugePages_Total:       0  
HugePages_Free:        0  
HugePages_Rsvd:        0  
HugePages_Surp:        0  
Hugepagesize:       2048 kB  
DirectMap4k:       53184 kB  
DirectMap2M:     2043904 kB  
  
## RHEL6  
  
RHEL6時代までのfreeコマンドでは、バッファ＆キャッシュ領域にスワップアウトしてしまうメモリも含まれているため、meminfoを直接参照していました。ただRHEL5以前とは異なり、RHEL6ではメモリ管理にSplitLRUが導入され、無名ページとファイルページを分て管理するようになっており、meminfoでもactive,inactive(メモリ回収候補ページのリスト)の登録ページ数が種別ごとに表示されるようになっているので、それを利用してメモリ計算をしています。  
  
参考：SplitLRU  
        https://www.linuxfoundation.jp/jp_uploads/seminar20090225/kosaki.pdf  
  
## freeコマンドとmeminfoの図解  
  
TODO  
  
【計算方法】  
  
/proc/meminfoのデータをawkでガリガリ計算させています。  
  
export LANG=C, LC_ALL=C  
  
awk '  
    BEGIN{  
        total=0;used=0;available=0;rate=0;  
    }  
  
    /^MemTotal:/         {total=$2;}  
    /^MemFree:/          {available=available+$2;}  
    /^Active\(file\):/   {available=available+$2;}  
    /^Inactive\(file\):/ {available=available+$2;}  
  
    END{  
        used=total-available;  
        rate=100*used/total;  
        printf("used%,total(KB),used(KB),available(KB)\n");  
        printf("%.1f,%d,%d,%d\n",rate,total,used,available);  
    }' /proc/meminfo;  
  
freeコマンド表示例  
  
#free  
             total       used       free     shared    buffers     cached  
Mem:       1020236     147168     873068        268      17824      54444  
 -/+ buffers/cache:      74900     945336  
Swap:      1048572          0    1048572  
  
/proc/meminfo表示例  
MemTotal:        1020236 kB  
MemFree:          873068 kB  
Buffers:           17832 kB  
Cached:            54460 kB  
SwapCached:            0 kB  
Active:            38724 kB  
Inactive:          51140 kB  
Active(anon):      17580 kB  
Inactive(anon):      256 kB  
Active(file):      21144 kB  
Inactive(file):    50884 kB  
Unevictable:           0 kB  
Mlocked:               0 kB  
SwapTotal:       1048572 kB  
SwapFree:        1048572 kB  
Dirty:                60 kB  
Writeback:             0 kB  
AnonPages:         17588 kB  
Mapped:            15584 kB  
Shmem:               268 kB  
Slab:              34304 kB  
SReclaimable:       8984 kB  
SUnreclaim:        25320 kB  
KernelStack:        1280 kB  
PageTables:         5236 kB  
NFS_Unstable:          0 kB  
Bounce:                0 kB  
WritebackTmp:          0 kB  
CommitLimit:     1558688 kB  
Committed_AS:     101760 kB  
VmallocTotal:   34359738367 kB  
VmallocUsed:      138664 kB  
VmallocChunk:   34359594564 kB  
HardwareCorrupted:     0 kB  
AnonHugePages:         0 kB  
HugePages_Total:       0  
HugePages_Free:        0  
HugePages_Rsvd:        0  
HugePages_Surp:        0  
Hugepagesize:       2048 kB  
DirectMap4k:        8192 kB  
DirectMap2M:     1040384 kB  
  
    追記(2015.9.19)  
        RHEL6.6にMemAvailableがバックポートされているという情報を頂いたので、こちらに整理しました。->【RHEL/CentOS】RHEL6.6にMemAbailableがバックポートされている件 - のぴぴのメモ  
  
## RHEL5以前  
  
で、一番悩ましいRHEL5以前です。この時代はfreeコマンドがダメでかつ、この当時は無名ページとファイルページを一緒に管理(一つのLRUリストにゴジャッと登録していた)ため、meminfoでもファイルページ数を的確に把握することができませんでした。  
  
そこでどうするかというと、ファイルキャッシュ＆バッファとしてカウントされるもののうち、無名ページに該当するものほとんどは、(1)tmpfs用のメモリ,(2)IPCの共有メモリ用のメモリ、の2つになることに着目します。そこで、それぞれの利用状況を取得してその分をmeminfoから取得した値から引き算し近似的に利用可能な容量を計算するというやり方をします。  
  
    キャッシュ＆バッファから除外する容量の計算方法  
        tmpfs --> dfコマンド結果から、tmpfsのusedの容量を足し算する  
        IPC共有メモリ --> "ipcs -u"コマンドの"pages allocated"(共有メモリ様に割り当て済みのページ数)を取得  
  
【freeコマンドとmeminfoの図解】  
  
TODO  
  
【計算方法】  
  
dfコマンド、"ipcs -u"コマンド、と/proc/meminfoのデータを組み合わせて、awkで計算させています。  
  
#!/bin/bash  
  
export LANG=C, LC_ALL=C  
  
#メモリファイル  
TMPFS=`/bin/df --portability |awk 'BEGIN{used=0;} /^tmpfs/{used=used+$3}END{print used}'`  
  
#IPC シェアードメモリ  
PAGE_SIZE=$((`/usr/bin/getconf PAGE_SIZE` / 1024))  
IPC_SHM=`/usr/bin/ipcs -u |awk -v SIZE=${PAGE_SIZE} '/^pages allocated/{print $3 * SIZE}'`  
  
#シェアードメモリ合計(tmpfs + ipc_shm)  
SHMEM=$(( ${TMPFS} + ${IPC_SHM} ))  
  
awk -v shmem=${SHMEM} '  
    BEGIN{  
        total=0;used=0;available=0;rate=0;  
    }  
  
    /^MemTotal:/ {total=$2;}  
    /^MemFree:/  {available=available+$2;}  
    /^Buffers:/  {available=available+$2;}  
    /^Cached:/   {available=available+$2;}  
  
    END{  
        available=available-shmem  
        used=total-available;  
        rate=100*used/total;  
        printf("used%,total(KB),used(KB),available(KB)\n");  
        printf("%.1f,%d,%d,%d\n",rate,total,used,available);  
    }' /proc/meminfo;  
  
freeコマンド表示例  
  
#free  
             total       used       free     shared    buffers     cached  
Mem:       2058764     313972    1744792          0      19748     165736  
  
    /+ buffers/cache:     128488    1930276  
  
Swap:      4095992          0    4095992  
  
/proc/meminfo表示例  
  
MemTotal:      2058764 kB  
MemFree:       1744792 kB  
Buffers:         19756 kB  
Cached:         165736 kB  
SwapCached:          0 kB  
Active:         103444 kB  
Inactive:       160096 kB  
HighTotal:           0 kB  
HighFree:            0 kB  
LowTotal:      2058764 kB  
LowFree:       1744792 kB  
SwapTotal:     4095992 kB  
SwapFree:      4095992 kB  
Dirty:             112 kB  
Writeback:           0 kB  
AnonPages:       78060 kB  
Mapped:          10520 kB  
Slab:            26440 kB  
PageTables:       3472 kB  
NFS_Unstable:        0 kB  
Bounce:              0 kB  
CommitLimit:   5125372 kB  
Committed_AS:   158884 kB  
VmallocTotal: 34359738367 kB  
VmallocUsed:    265472 kB  
VmallocChunk: 34359470723 kB  
HugePages_Total:     0  
HugePages_Free:      0  
HugePages_Rsvd:      0  
Hugepagesize:     2048 kB  
  
蛇足  
その1：無名ページとファイルページ  
  
まず、このページではファイルページと記載しましたが、正確には「ファイルバックド(file-backed)ページ」ですかね。  
また、ファイルページを記事内では「ディスクと同期して解放」と記載してますが、正確にはディスクと同期するのは未同期データの解放の時のみです。同期済みの場合は同期処理なく解放します。  
  
無名ページとファイルページの違いをざっくりと書くと、本文にも記載しましたが以下の通りになります。  
  
    無名ページ --->メモリが足りない時スワップアウト(スワップへ退避)され、物理メモリから解放されるページ  
    ファイルページ ---> メモリが足りない時ディスク同期し解放されるページ  
  
蛇足になりますが、10年前にカーネル本(オライリー＆ペンギン黒本)を読んだ時は、いきなり「無名ページ」と出て来て、何が何だか分からず挫折した経験があります。  
それから10年経ってやっと「スワップする/しない、の差なのね」という理解に到達。この辺の理解している人としてない人の溝の深さが、カーネルを理解する上での障壁(カーネル難しぃ〜)の一つになっているのかなと思います。  
その２：図解の内容のツッコミ  
  
「SReclaimable」領域を、「ディスク同期して解放」で色塗りしていますが、実際は単純にslabとして未使用領域で、メモリが足りない場合は単純に開放するだけですね。  
後で気付いたのですが、図を修正するのが面倒なので、そのままにしちゃいました。  
その３：RHEL6の計算  
  
RHEL6では、"active(file)"と"inative(file)"から計算していますが、RHEL7のavailaleのカーネル内部の計算を見るに、"SReclaimable"も加算しなければならないですね。  
まあ全体量に対して、"SRelaimale"は小さいので加算しなくても大した影響はないとおもいますが。  
その４：Inactiveを空き領域とすることは間違い。  
  
ちなみに、linuxでのメモリ利用量(空き容量)をググると「利用可能なメモリ ≒（MemFree＋Inactive）」という話がでますが、これはガチャピン先生が指摘されていますが、間違いです。
