---
title: "マルチメディア: 画像"
slug: Learn_web_development/Extensions/Performance/Multimedia
original_slug: Learn/Performance/Multimedia
l10n:
  sourceCommit: 0a9bfb1aee6b1b88c14acd3f66501cc01320f184
---

{{LearnSidebar}}{{PreviousMenuNext("Learn/Performance/measuring_performance", "Learn/Performance/video", "Learn/Performance")}}

画像や動画といったメディアは、平均的なウェブサイトでダウンロード量の 70% 以上を占めています。ダウンロードのパフォーマンスという観点からは、メディアを減らし、ファイルサイズを縮小することは、低いハードルであると言えます。この記事では、ウェブパフォーマンスを向上させるために画像や動画を最適化することについて見ていきます。

<table>
  <tbody>
    <tr>
      <th scope="row">前提条件:</th>
      <td>
        基本的なコンピューターリテラシー、
        <a
          href="/ja/docs/Learn/Getting_started_with_the_web/Installing_basic_software"
          >基本的なソフトウェアのインストール</a
        >、
        <a href="/ja/docs/Learn/Getting_started_with_the_web"
          >クライアント側のウェブ技術</a
        >の基本的な知識
      </td>
    </tr>
    <tr>
      <th scope="row">目標:</th>
      <td>
        さまざまな画像形式とそのパフォーマンスへの影響、そして画像がページ全体の読み込む時刻に与える影響を縮小する方法について学ぶ。
      </td>
    </tr>
  </tbody>
</table>

> [!NOTE]
> これは、ウェブ上でのマルチメディア配信を最適化するための高レベルな入門書であり、一般的な原則とテクニックに応じたものです。より詳細なガイドについては、 <https://web.dev/learn/images> を参照してください。

## なぜマルチメディアを最適化するのか

平均的なウェブサイトでは、[帯域幅の 51% が画像、続く 25% が動画](https://discuss.httparchive.org/t/state-of-the-web-top-image-optimization-strategies/1367)であり、マルチメディアコンテンツに対応し最適化することが重要であると言ってもよいでしょう。

データ使用量に配慮する必要があります。多くの人は、上限付きのデータプラン、あるいは文字通りメガバイト単位で支払う従量制のデータプランを利用しています。これは新興市場の問題でもありません。2018 年現在、[イギリスの 24% がまだ従量制課金を使用しています](https://www.ofcom.org.uk/__data/assets/pdf_file/0021/113169/Technology-Tracker-H1-2018-data-tables.pdf)。

また、多くのモバイル端末では RAM に制限があるため、メモリーにも配慮する必要があります。画像がダウンロードされると、メモリーに格納される必要があることを覚えておくことが重要です。

## 画像配信の最適化

帯域幅を最も消費するにもかかわらず、画像のダウンロードが[知覚的パフォーマンス](/ja/docs/Learn_web_development/Extensions/Performance/Perceived_performance)に与える影響は、多くの人が予想するよりはるかに小さいです（主に、ページのテキストコンテンツはすぐにダウンロードされ、ユーザーは到着時に画像のレンダリングを見ることができるからです）。しかし、使い勝手を良くするためには、訪問者ができるだけ早く見ることができることが重要であることに変わりはありません。

### 読み込み戦略

多くのウェブサイトにおける最大の改善点のひとつは、画像を見える直前に[遅延読み込み](/ja/docs/Web/Performance/Guides/Lazy_loading)することで、訪問者がスクロールして見るかどうかにかかわらず、最初のページ読み込み時にすべてダウンロードしないようにすることです。多くの JavaScript ライブラリー、たとえば [lazysizes](https://github.com/aFarkas/lazysizes) などで、これを実装することができます。また、ブラウザーベンダーは、現在実験段階であるネイティブの `lazyload` 属性に取り組んでいます。

画像のサブセットを読み込むだけでなく、画像そのものの形式も見ていく必要があります。

- 最適なファイル形式を読み込んでいますか？
- 画像はしっかり圧縮してありますか？
- 正しいサイズを読み込んでいますか？

#### 最も適した形式

最適なファイル形式は、通常、画像の性格によって異なります。

> [!NOTE]
> 画像の種類に関する一般的な情報は、[画像ファイルの種類と形式ガイド](/ja/docs/Web/Media/Guides/Formats/Image_types)を参照してください。

[SVG](/ja/docs/Web/Media/Guides/Formats/Image_types#svg_scalable_vector_graphics) 形式は、色が少なく、写真のようにリアルでない画像に適しています。この場合、ソースがベクターグラフィック形式で利用できることが必要です。そのような画像がビットマップとしてしか存在しない場合は、[PNG](/ja/docs/Web/Media/Guides/Formats/Image_types#png_portable_network_graphics) が予備のフォーマットとして選ばれます。この種類のモチーフの例としては、ロゴ、イラスト、図表、アイコンなどがあります（メモ: SVG はアイコンフォントよりもはるかに優れています！）。どちらの形式も透過に対応しています。

PNG は 3 種類の出力の組み合わせで保存することができます。

- 24 ビットカラー＋8 ビット透過率 - サイズを犠牲にして、完全な色精度と滑らかな透過率を提供します。この組み合わせは避け、WebP（下記参照）にした方がよいでしょう。
- 8 ビットカラー＋8 ビット透過率 - 255 色を超えることはできないが、滑らかな透過率を維持することができます。サイズはあまり大きくありません。これらは、おそらく使いたい PNG です。
- 8 ビット色＋1 ビット透過率 - 255 色を超えることはできず、ただピクセルごとに非透過または完全透過のどちらかだけを提供するため、透過率の境界線が硬くギザギザに見えます。サイズは小さいが、視覚的な忠実さが代償となります。

SVGを最適化するための良いオンラインツールとしては、[SVGOMG](https://jakearchibald.github.io/svgomg/) があります。PNG の場合は、[ImageOptim online](https://imageoptim.com/online) や [Squoosh](https://squoosh.app/) があります。

透過のない写真モチーフの場合、選べる形式の範囲はかなり広くなります。安全性を重視するならば、圧縮率の高い**プログレッシブ JPEG** をお勧めします。プログレッシブ JPEG は、通常の JPEG とは異なり、プログレッシブにレンダリングします（そのため、この名前がついています）。つまり、画像が上から下まで完全な解像度で読み込まれたり、ダウンロードが完了したりしてから表示されるのではなく、ユーザーには低解像度のバージョンが表示され、ダウンロードが進むにつれて鮮明になっていく様子が見えます。このような場合に適した圧縮ソフトは MozJPEG で、例えばオンライン画像最適化ツール [Squoosh](https://squoosh.app/) で使用することができます。品質設定は 75% で、適切な結果が得られるはずです。

他にも、圧縮に関して JPEG より能力を上回る形式がありますが、すべてのブラウザーで利用できるわけではありません。

- [WebP](/ja/docs/Web/Media/Guides/Formats/Image_types#webp_画像) — 画像とアニメーション画像の両方に最適な選択肢です。WebP は PNG や JPEG よりもはるかに優れた圧縮率を持ち、より高い色深度、アニメーションフレーム、透過率などに対応しています（ただし、プログレッシブ表示には対応していません）。macOS デスクトップ Big Sur の Safari 14 以前を除く、主要なブラウザーで対応しています。

  > [!NOTE]
  > Apple が [Safari 14 で WebP の対応を発表](https://developer.apple.com/videos/play/wwdc2020/10663/?time=1174)しているにもかかわらず、 Safari のバージョン 16.0 より前では、 macOS の 11/Big Sur より前のデスクトップ版では正常に `.webp 画像が表示されなせん、 iOS 14 の Safari では `.webp` 画像が正しく表示されます。

- [AVIF](/ja/docs/Web/Media/Guides/Formats/Image_types#avif_画像) — 高性能でロイヤリティフリーの画像形式であるため、画像とアニメーション画像の両方に適しています（WebP よりもさらに効率的ですが、対応はそれほど広くありません）。これで Chrome、Opera、Firefox で対応しています。[以前の画像形式を AVIF に変換するオンラインツール](https://avif.io/)も参照してください。
- **JPEG2000** — かつては JPEG の後継とされていましたが、Safari でのみ対応しています。プログレッシブ表示にも対応していません。

JPEG-XR と JPEG2000 の対応も狭く、デコードするコストも考慮すると、JPEG の対抗馬は WebP しかありません。そのため、画像も WebP で提供することができます。これは、`<picture>` 要素で [type 属性](/ja/docs/Web/HTML/Reference/Elements/picture#the_type_attribute)を備えた `<source>` 要素により補助することで実現することができます。

もし、この作業がすべて複雑で、チームにとって負担が大きいと感じるのであれば、画像 CDN として使用できるオンラインサービスもあります。このサービスは、画像をリクエストされた機器や ブラウザーの種類に応じて、その場で正しい画像形式を自動配信してくれます。[Cloudinary](https://cloudinary.com/blog/make_all_images_on_your_website_responsive_in_3_easy_steps) や [Image Engine](https://imageengine.io/) が代表的なものです。

最後に、ページにアニメーション画像を掲載するために、Safari では `<img>` と `<picture>` 要素に動画ファイルを使用することができることをご存知でしょうか？他にも現代のブラウザーでは、**アニメーション WebP** を追加することができます。

```html
<picture>
  <source type="video/mp4" src="giphy.mp4" />
  <source type="image/webp" src="giphy.webp" />
  <img src="giphy.gif" alt="A GIF animation" />
</picture>
```

#### 最適なサイズでの提供

画像配信では、「ひとつのサイズですべてをカバーする」手法は最良の結果をもたらしません。つまり、小さな画面にはより小さな解像度の画像を、大きな画面にはその逆の画像を配信したいものです。また、高 DPI 画面（例えば "Retina"）の機器には、より高解像度の画像を提供したいものです。そのため、中間画像のバリエーションをたくさん作成するだけでなく、正しいファイルを正しいブラウザーに提供する方法も必要です。そこで、`<picture>` と `<source>` 要素に [media](/ja/docs/Web/HTML/Reference/Elements/source#media) や [sizes](/ja/docs/Web/HTML/Reference/Elements/source#sizes) 属性を付けてアップグレードする必要があるのです。これらの属性をすべて組み合わせる方法についての詳しい記事は[こちら](https://www.smashingmagazine.com/2014/05/responsive-images-done-right-guide-picture-srcset/)にあります。

高解像度画面に関して、興味深い効果が 2 つあります。

- 高 DPI 画面では、人間は圧縮による劣化を感じにくくなるため、このような画面向けの画像では、通常よりも圧縮率を上げることができます。
- [2x を超える DPI の解像度の上昇を見抜ける人はとても少ない](https://observablehq.com/@eeeps/visual-acuity-and-device-pixel-ratio)ため、2x を超える解像度の画像を提供する必要はないということです。

#### ダウンロードする画像の優先順位（順番）の制御

重要度の高い画像を、重要度の低い画像よりも早く取得することで、知覚的パフォーマンスを向上させることができます。

最初に確認することは、コンテンツ画像に `<img>` 要素または `<picture>` 要素を使用しているか、背景画像を CSS の `background-image` で定義しているかということです。`<img>` 要素で参照する画像は、背景画像より高い読み込み優先度が割り当てられます。

2 つ目は、優先度ヒントの採用により、画像タグに `fetchPriority` 属性を追加することで、さらに優先度を制御できるようになることです。画像に優先度ヒントを使用する例としては、最初の画像が後続の画像よりも高い優先度を持つカルーセルが挙げられます。

### レンダリング戦略: 画像を読み込むときのジャンクの防止

画像は非同期で読み込まれ、最初の描画の後も読み込まれ続けるため、読み込む前にサイズを定義しておかないと、ページコンテンツに再フローを発生させてしまうことがあります。例 えば、画像の読み込みによってテキストがページ下に押し出されるような場合です。このため、`width` と `height` 属性を設定し、ブラウザーがレイアウトにそれらのための空間を確保できるようにすることが重要です。

画像の `width` と `height` 属性を HTML の {{htmlelement("img")}} 要素に明記すると、画像が読み込まれる前にブラウザーがその画像のアスペクト比を計算することができます。このアスペクト比は、画像を表示するために必要な空間を確保するために使用され、画像がダウンロードされ画面に描画される際のレイアウトのずれを縮小したり、防止したりすることができます。レイアウトのずれを縮小することは、使い勝手や ウェブパフォーマンスを向上させるための重要な要素です。

ブラウザーは、HTML が解釈されるとコンテンツのレンダリングを開始しますが、多くの場合、画像を含むすべての資産がダウンロードされる前にレンダリングを開始します。サイズを記載することで、ブラウザーは、ページを最初にレンダリングする際に画像が読み込まれる際に、各画像を配置するための正しいサイズのプレースホルダーボックスを確保することができます。

![2 つのスクリーンショットで、最初のものは画像はなく空間が確保されており、2 つ目は確保された空間に画像が読み込まれた状態です。](ar-guide.jpg)

`width` と `height` 属性がないと、プレースホルダーの空間が作成されないため、ページがレンダリングされた後に画像が読み込まれると、{{glossary('jank', 'ジャンク')}}つまりレイアウトのずれが目立つようになります。ページの再フローや再描画は、パフォーマンスやユーザビリティの課題です。

レスポンシブデザインにおいて、コンテナーが画像より狭い場合、画像がコンテナーからはみ出さないようにするために、以下の CSS を使用するのが一般的です。

```css
img {
  max-width: 100%;
  height: auto;
}
```

レスポンシブレイアウトには便利ですが、幅と高さの情報が記載されていない場合、ジャンクの原因となります。`<img>` 要素が解釈されたときに高さの情報がない場合、画像を読み込む前に、この CSS は高さを 0 に設定します。ページが画面の内側へ最初に描画された後に画像が読み込まれると、新たに決めた高さを確保するために、ページは再フローや再描画を行い、レイアウトが変化します。

ブラウザーには、実際の画像が読み込まれる前に、画像のサイズを調整する仕組みがあります。 `<img>`、`<video>`、`<input type="button">` 要素に `width` と `height` 属性が設定されている場合、その縦横比は読み込む前に計算され、提供された寸法をブラウザーが利用できるようになります。

アスペクト比は次に高さの計算に用いられ、正しいサイズが `<img>` 要素に適用されます。つまり、画像が読み込まれる際に前述のジャンクは発生しないか、指定されている寸法が完全に正確でない場合でも、最小限に抑えられます。

アスペクト比は、画像の読み込み時にのみ空間を確保するために使用します。画像が読み込まれると、属性によるアスペクト比ではなく、読み込まれた画像の固有のアスペクト比が使用されます。これにより、属性の寸法が正確でない場合でも、確実に正しいアスペクト比で表示されます。

過去 10 年来、開発者は `width` と `height` 属性を HTML の {{htmlelement("img")}} の画像では省略することが最善の手法として推奨されてきましたが、アスペクト比の割り当てのため、これら 2 つの属性を指定することが開発者の最善の手法と考えられています。

背景画像については、 `background-color` 値を設定することが重要で、そうすれば画像がダウンロードされる前に、その上に重なるコンテンツが読めるようになります。

## まとめ

この章では、画像の最適化について見ていきました。これで、平均的なウェブサイトの平均帯域幅の合計の半分を最適化する方法について、全般的に理解したことになります。これは、ユーザーの帯域幅を消費し、ページ読み込みを遅くするメディアの種類の 1 つに過ぎません。次に、帯域幅消費の 20% を占める動画の最適化について見ていきましょう。

{{PreviousMenuNext("Learn/Performance/measuring_performance", "Learn/Performance/video", "Learn/Performance")}}
