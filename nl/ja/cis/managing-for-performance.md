---
  
copyright:
   years: 2018
lastupdated: "2018-03-16"
 
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}


# 最高のパフォーマンスを実現するための CIS デプロイメントの管理

IBM Cloud Internet Services (CIS) は、画像を最適化し、Web コンテンツをエンド・ユーザーにできる限り近い場所に保存することで、お客様の顧客に最速のエクスペリエンスを提供できます。お客様のコンテンツは、プロキシー・エッジ・サーバーから読み込まれます (これにより、遅延を低減します)。

IBM CIS では、Web コンテンツの読み込みを加速するベスト・プラクティスを使用して、サイトのパフォーマンスをさらに高めることができます。ここでは、CIS で Web コンテンツのパフォーマンスを向上させるためのベスト・プラクティスについて具体的に説明します。

**推奨事項とベスト・プラクティス**

 * 静的および準静的な Web コンテンツをできる限り多くキャッシュする
 * イベント・ドリブン・コンテンツについては、API を使用してキャッシュをパージする
 
## ベスト・プラクティス 1: 静的および準静的な Web コンテンツをできる限り多くキャッシュする

  * 静的な HTML Web ページには、**「すべてキャッシュする (Cache Everything)」**を有効にする
  * 時々変更するコンテンツには、従来の**「存続時間 (TTL)」**を使用する

### 時々変更するコンテンツには、従来の TTL (存続時間) を使用する
コンテンツがめったに変わらない場合は、従来の TTL を設定してできる限り多くのキャッシュを利用します。再検証要求の割合が高い場合は、顧客に悪影響を与えることなくコンテンツの TTL を増やすことができます。キャッシュを効果的に使用すると、再検証の頻度が少なくなるので、パフォーマンスが向上します。

### アイテムがキャッシュされているかどうかを調べる方法
IBM CIS はオブジェクトをキャッシュしようとするときに応答ヘッダー `CF-Cache-Status` を追加します。キャッシングが正常に行われると、状態を示す次のキーワードのいずれかがこのヘッダーの値に設定されます。

* **MISS:** 資産はまだキャッシュに存在していませんでした。または TTL の期限が切れていました (つまり、キャッシュ制御の最大経過時間 0 に達していました)。
* **HIT:** 資産はキャッシュから配信されました。
* **EXPIRED:** この資産はキャッシュから配信されましたが、次の要求には再検証が必要になります。
* **REVALIDATED:** 資産はキャッシュから配信されました。TTL の期限は切れていましたが、オリジンへの `If-Modified-Since` 要求によって、資産が変更されていないことが判明しました。したがって、キャッシュにあるバージョンが再度有効と見なされました。

## ベスト・プラクティス 2: イベント・ドリブン・コンテンツについては、API を使用してキャッシュをパージする
例えば、新しい投稿がブログに追加されるたびに、API コマンドを使用して簡単に CIS キャッシュをパージできます。イベント・ドリブン・コンテンツはよく見られます。IBM は、古いコンテンツをお客様のユーザーに配信しないように保証することを簡単にしました。グローバル・ネットワーク全体でただちにキャッシュをパージするコマンドについては次でリストします。お客様は IBM のキャッシング・アプリケーションを使用することも、API を使用することもできます。

  * キャッシュ・タグを使用してキャッシュをパージする
  * グローバルにキャッシュをパージする
  * ページ・ルールによってキャッシュをパージする
  * 高度なキャッシング機能を使用する

### キャッシュ・タグを使用してキャッシュをパージする
キャッシュ・タグを使用すると、パージするコンテンツのバケットを定義できます。これは、一緒に変更されることが多いオブジェクトをまとめることができる優れた方法です。例えば、HTML ブログの投稿とその画像コンテンツのすべてを一緒にタグ付けできます。モバイル専用コンテンツもキャッシュ・タグを使用してまとめられるので、モバイル・ドメインに新しい更新をプッシュするときにすべてをパージできます。

### グローバルにキャッシュをパージする
キャッシュ全体を強制的に再検証するという選択肢もあります。キャッシュに保管されているすべてのオブジェクトをリセットして、すべての要求がオリジン・サーバーに転送されるようにすることができます。

### ページ・ルールによってキャッシュをパージする
ページ・ルールでは、正規表現に基づいてキャッシュ全体をパージできます。事前定義されたページ・ルールを使用し、そのページ・ルールと照らしてすべてのヒットを再検証できます。1 ページあたり最大 50 個のページ・ルールを作成できます。

### 高度なキャッシング機能を使用する

**Cookie のキャッシュをバイパス (Bypass Cache on Cookie):** ページ・ルールでこの機能を設定すると、特定の名前の Cookie が存在しない限り、キャッシュにあるオブジェクトを提供できます。例えば、顧客がログインしていることを示す `SessionID` Cookie が検出されたので、パーソナライズされたコンテンツを表示しなければならないという場合を除き、ホーム・ページのキャッシュ・バージョンを提供したりできます。
