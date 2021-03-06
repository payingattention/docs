# ブロックチェーンのビザンチンフォールトトレランス

Erik Zhang

erik@vcage.com

## 要約

​この論説では、ブロックチェーンシステムのために改良されたビザンチンフォールトトレランスアルゴリズムについて述べます。仮定として、このシステムでは、メッセージは紛失、損傷、遅延、および繰り返しを受ける可能性があります。また、送信順序は必ずしもメッセージの受信順序と一致するとは限りません。ノードの活動は任意であり、いつでもネットワークに参加、退出することが可能です。（情報を破棄や改竄したり、単に活動を中止することもあります）。人為的なグリッチと非人為的なグリッチのどちらも発生することがあります。我々のアルゴリズムは、n個のノードを含むコンセンサスシステムに対して、𝑓 = ⌊ (𝑛−1) / 3 ⌋のフォールトトレランスを提示します。このトレランスキャパシティは、セキュリティとユーザビリティを確保し、あらゆるネットワーク環境に適しています。

## 概要

ブロックチェーンは、分散型の台帳システムです。デジタル化された資産、財産権証明書、クレジットポイントなどの登録と発行に使用することができます。これにより、ピアツーピアでの転送、支払い、取引が可能になります。ブロックチェーンの技術は、ビットコインを代表とするような暗号学のメーリングリスト内において、Satoshi Nakamoto氏によって提示されました。それ以降、e-キャッシュシステム、株式交換、スマートコントラクトシステムなど、ブロックチェーンに基づく多くのアプリケーションが登場しました。

ブロックチェーンシステムは、従来の中央集権型の台帳システムに比べて、透明性、不変性、2重支払いへの耐性という特徴を持ち、信頼できる第三者に依存していません。

しかし、すべての分散システムと同様に、ブロックチェーンシステムは、ネットワーク遅延、伝送エラー、ソフトウェアのバグ、セキュリティホール、ブラックハットハッカーの脅威を克服する必要があります。さらに、分散化された性質上、システム参加者が信頼できないことを示唆しています。悪意のあるノードが出現する可能性があるため、利害の不一致の結果、データに差異が生まれる可能性があります。

これらの潜在的な問題に対処するために、ブロックチェーンシステムでは、すべてのノードが認識された総台帳のコピーを持つことを保証するための効率的なコンセンサスメカニズムが必要です。従来のフォールトトレランスメカニズムは、分散システムとブロックチェーンシステムが直面する問題に完全に対応することはできない可能性があります。そのため、universal cure-to-all fault tolerance solutionが必要です。

ビットコインによって採用されているProof of Workのメカニズム[1]は、この問題をかなり巧妙に扱っています。しかしながら、このProof of Workでは膨大なコスト、すなわち著しい電力コストとエネルギー消費が伴います。さらに、ビットコインの存在により、新しいブロックチェーンは、コンピューテーショナルアタックを防ぐために、異なるハッシュアルゴリズムを見つけなければなりません。たとえば、LitecoinはBitcoinのSHA256ではなくSCRYPTを採用しています。

ビザンチンフォールトトレランスメカニズムは分散システムの普遍的な解決策[5]です。この論説では、1999年にCastroとLiskovによって提案された実用的ビザンチンフォールトトレランス（PBFT）[3]に基づいて、ブロックチェーンシステムのための改良ビザンチンフォールトトレランスアルゴリズムが提示されています。

## システムモデル

ブロックチェーンは、参加者がピアツーピアネットワークを介して相互に接続する分散型台帳システムです。その中のすべてのメッセージはブロードキャストで送信されます。ノードには通常のノードとブックキーパーノードの2種類の役割があります。通常のノードは、システムを使用して転送や交換を行い、台帳データを受け取ります。一方、ブックキーパーノードはネットワーク全体の会計業務を提供し、元帳を維持します。

仮定として、このシステムでは、メッセージは紛失、損傷、遅延、および繰り返しを受ける可能性があります。また、送信順序は必ずしもメッセージの受信順序と一致するとは限りません。ノードの活動は任意であり、いつでもネットワークに参加、退出することが可能です。（情報を破棄や改竄したり、単に活動を中止することもあります）。人為的なグリッチと非人為的なグリッチのどちらも発生することがあります。

情報送信の完全性と真正性は暗号で保証されると同時に、送信者は送信されたメッセージのハッシュ値に署名を添付する必要があります。ここでは〈𝑚〉<sub>𝜎𝑖</sub>をノードiから送られたｍのデジタル署名というメッセージと定義し、D(m)はメッセージmのハッシュ値と定義します。特別な説明がない限り、この記事で言及しているすべての署名は、メッセージのハッシュ値に対する署名です。

## アルゴリズム

​私たちのアルゴリズムは、セキュリティとユーザビリティを確保します。コンセンサスを誤ったノードが⌊ (𝑛−1) / 3 ⌋を超えないようにすることで、システムの機能性と安定性が保証されます。𝑛 = |𝑅|はコンセンサスに参加するノードの総数を表し、Rはそのコンセンサスノードの集合体を意味します。𝑓 = ⌊ (𝑛−1) / 3 ⌋とすると、fはシステム内で許容される誤ったノードの最大数を表します。実際、総台帳はブックキーパーノードによって維持され、通常ノードはコンセンサスに参加しません。これは、全体の合意形成手続きを示すためです。

すべてのコンセンサスノードは、現在のコンセンサスの状態を記録するために状態テーブルを維持する必要があります。コンセンサスの開始から終了までに用いられるデータセットはビューと呼ばれます。現在のビュー内でコンセンサスに到達できない場合は、ビューの変更が必要です。各ビューは0から始まる数字vで識別され、コンセンサスが達成されるまで増加します。

​各コンセンサスノードは、0から始まり、最後のノードはn − 1の番号が付けられます。各ラウンドのコンセンサスの形成では、一つのノードが、議長であるスピーカーの役割となり、他のノードが議員の役割になります。スピーカーの番号pは、次のアルゴリズムによって決定されます：現在のブロックの高さをhと仮定すると、𝑝 = (ℎ − 𝑣) 𝑚𝑜𝑑 𝑛、pの値の範囲は0 ≤ 𝑝 < 𝑛になります。

​コンセンサスの各ラウンドごとに、ブックキーパーノードから少なくとも𝑛 − 𝑓の数の署名を伴った新しいブロックが生成されます。ブロックが生成されると、コンセンサス形成の新しいラウンドが開始され、v = 0にリセットされます。

### 通常の手順

ブロック生成の時間間隔をtと設定すると、アルゴリズムは通常の状況下で次の手順で実行されます。

1) ノードは、送信者署名付きのトランザクションデータをネットワーク全体に配信します。

2) すべてのブックキーパーノードは、トランザクションデータの配信を独立して監視し、そのデータを各々のメモリに格納する。

3) t時間経過後、スピーカーは〈𝑃𝑒𝑟𝑝𝑎𝑟𝑒𝑅𝑒𝑞𝑢𝑒𝑠𝑡,ℎ,𝑣,𝑝,𝑏𝑙𝑜𝑐𝑘,〈𝑏𝑙𝑜𝑐𝑘〉<sub>𝜎𝑝</sub>〉を送信します。

4) 提案を受け取った後、議員iは〈𝑃𝑒𝑟𝑝𝑎𝑟𝑒𝑅𝑒𝑠𝑝𝑜𝑛𝑠𝑒,ℎ,𝑣,𝑖,〈𝑏𝑙𝑜𝑐𝑘〉<sub>𝜎𝑖</sub>〉を送信します。

​5) ノードは、少なくとも𝑛 − 𝑓  〈𝑏𝑙𝑜𝑐𝑘〉<sub>𝜎𝑖</sub>𝑖, のコンセンサスに達し、フルブロックを公開する。

6) ノードは、フルブロックを受け取った後、現在のトランザクションをメモリから削除し、次のラウンドのコンセンサスを開始する。

すべてのコンセンサスノードにおいて、少なくとも𝑛 − 𝑓ノードは同じ元の状態にあることが必要である。つまり、すべてのノードiにおいて、ブロックの高さhとビュー番号vは同じです。これは難しいことではなく、ビューの変更によってvの一貫性を保ちながらブロックを同期させることにより、hの一貫性を得ることができます。ブロック同期についてはこの論説では扱いません。ビューの変更については、次のセクションを参照してください。

配信を監視し、提案を受信した後、ノードはトランザクションを検証するすることになります。後者が公開されると、メモリ内に違法なトランザクションを書き込むことはできません。提案の中に違法なトランザクションが含まれている場合、このコンセンサスのラウンドは放棄され、直ちにビューの変更が行われます。検証手順は次のとおりです。

1) トランザクションのデータ形式はシステムルールと一貫していますか？そうでない場合、取引は違法になります。

2) トランザクションはすでにブロックチェーンに入っていますか？はいの場合、取引は違法になります。

3) トランザクションのすべてのコントラクトスクリプトが正しく実行されていますか？そうでない場合、取引は違法になります。

4) トランザクションに複数の支払がありますか？はいの場合、取引は違法になります。

5) 上記の手順で取引が違法とされなかった場合、それは合法であると判断されます。

#### View Change

2<sup>𝑣+1</sup> ⋅ 𝑡時間のインターバルの後に、ノードiが合意に達することができないか、違法なトランザクションが含まれている提案を受けなければならない場合、ビューの変更が行われます。

1) k=1、𝑣<sub>𝑘</sub> = 𝑣 + 𝑘が与えられる。

2) ノードiはビューの変更要求を送信します。〈𝐶ℎ𝑎𝑛𝑔𝑒𝑉𝑖𝑒𝑤,ℎ,𝑣,𝑖,𝑣<sub>𝑘</sub>〉

3) いづれかのノードが少なくとも𝑛 − 𝑓の同一のv<sub>k</sub>を異なるiから受信すると、ビューの変更は完了します。
𝑣 = 𝑣<sub>𝑘</sub>を設定し、コンセンサス形成が始まります。

4) もし2<sup>𝑣+1</sup> ⋅ 𝑡時間の後に、ビューの変更が完了していない場合、kが増加し、手順2)に戻ります。

kの増加に伴って、待機の超過時間が指数関数的に増加するため、頻繁なビューの変更は回避され、ノードはvに対して一貫性にを保つように促されます。

ビューの変更が完了するまでは元のビューvは有効なので、ネットワークの遅延に起因する不必要なビューの変更を回避できます。

### フローチャート

![](~/assets/consensus_flowchart.jpg)

## フォールトトレランスキャパシティ

​私たちのアルゴリズムは、n個のノードを含むコンセンサスシステムに対して、𝑓 = ⌊ (𝑛−1) / 3 ⌋のフォールトトレランスを提示する。このトレランスキャパシティには、セキュリティとユーザビリティを確保し、あらゆるネットワーク環境に対応しています。

ノードからの要求データには送信者の署名が含まれているため、悪意のあるブックキーパーノードは要求を改竄できません。代わりに、システムステータスを過去に戻して、システムを強制的にフォークさせようとします。

仮定として、システムのネットワーク環境において、コンセンサスノードは3つの部分に分けられます：𝑅 = 𝑅1 ∪ 𝑅2 ∪ 𝐹、𝑅1 ∩ 𝑅2 = ∅、𝑅1 ∩ 𝐹 = ∅、𝑅2 ∩ 𝐹 = ∅。また、仮定として、
R1とR2の両方は、インフォメーションサイロ内の誠実なブックキーパーノードであり、それらはセット内のノードとしか通信できません。Fはすべての悪意のあるノードです。さらに、Fのネットワーク状態は、それらがR1およびR2を含む任意のノードと通信すること可能です。
Fがシステムをフォークしたい場合は、R1とコンセンサスに達し、ブロックを公開し、
R2に通知せずに第2のコンセンサスに到達し、R1のコンセンサスを取り消す必要があります。
​これを達成させるには、|𝑅1| + |𝐹| ≥ 𝑛 − 𝑓および|𝑅2| + |𝐹| ≥ 𝑛 − 𝑓である必要がある。
最悪のシナリオでは|𝐹| = 𝑓​、すなわち 悪意のあるノードの数は、システムが許容できる最大値になります（前述の関係が|𝑅1| ≥ 𝑛 − 2𝑓​と​|𝑅2| ≥ 𝑛 − 2𝑓）。上記二つを合わせると、​𝑛 ≤ 3𝑓として簡略化することができ、|𝑅1| + |𝑅2| ≥ 2𝑛 − 4𝑓​となる。前者と矛盾する𝑓 = ⌊ (𝑛−1) / 3 ⌋が与えられた場合、システムはトレランスの範囲内でフォークできないことが証明できます。

## 参考文献

[1] Nakamoto S. Bitcoin: A peer-to-peer electronic cash system[J]. 2008.

[2] Lamport L, Shostak R, Pease M. The Byzantine generals problem[J]. ACM Transactions on Programming Languages and Systems (TOPLAS), 1982, 4(3): 382-401.

[3] Castro M, Liskov B. Practical Byzantine fault tolerance[C]//OSDI. 1999, 99: 173 186.

[4] Bracha G, Toueg S. Asynchronous consensus and broadcast protocols[J]. Journal of the ACM (JACM), 1985, 32(4): 824-840.

[5] 范捷, 易乐天, 舒继武. 拜占庭系统技术研究综述[J]. 软件学报, 2013, 6: 012.

