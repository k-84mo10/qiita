---
title: Chocolate Journeyに挑戦してみた！！
tags: ["アルゴリズム", "二分探索", "C++"]
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
この記事は、[EEIC Advent Calendar 2023](https://qiita.com/advent-calendar/2023/eeic)の16日日、12/16の記事として書かれています。
他の記事の方もよろしくお願いします。

# はじめに
もうすぐクリスマスも近づいてきましたね！！
この記事を読んでいる皆様も、「クリスマスは恋人とどんなデートしようかな？」と考えているのではないでしょうか？
この記事ではデートの定番である「カフェデート」にオススメなカフェについて紹介します。
今回私が訪れたのは、バニラビーンズみなとみらい本店。

https://tabelog.com/kanagawa/A1401/A140103/14053382/dtlmenu/

チョコレート専門店ということで、チョコレート好きの方にはたまらないカフェです。
まず入口を入ると、チョコレート売り場が、沢山の可愛らしいチョコレートが並んでいます。
そして、奥にはカフェスペースがあり、チョコレートを使ったスイーツやコーヒーが楽しめます。
今回、私が注文したのはバニラビーンズの名物「Chocolate Journey」です。
Chocolate Jouneyとは、13種類のチョコレートの味の違いを楽しむことができるスイーツです。
そして、そのうち一つは「利きチョコレート」。
このメニューの醍醐味はこの利きチョコがどのチョコレートなのか、を当てるゲームです。

# Chocolate Jouneyのルール
Chocolate Jouneyのルールを詳細に説明します。
Chocolate Jouneyは13種類のチョコレートが1粒ずつ並べられたプレートが提供されます。
プレートのチョコレートは左から以下の種類となってます。
1. Colombia 55
2. Dominica 60
3. Brazil 65
4. Haiti 68
5. Colombia 70
6. Belize 70
7. Trinidad 77
8. Ghana 80
9. Today's Chocolate
10. Minatomirai Blend
11. Berry Berry
12. Caramel
13. Cookies & Nibs

"Today's Chocolate"は利きチョコ、すなわち今回のターゲットで、1から8のチョコレートの中から1つ選ばれます。
つまり、このゲームの目的は、"Today's Chocolate"がどのチョコか当てることです。
（10以降のチョコレートはゲームには関係ありません。）
様々なチョコレートを味わいながら、味覚勝負をする、というのがこのゲームの醍醐味です。
なかなか楽しそうですね！！

# 作戦を考えよう！！
さて、やるからには利きチョコをちゃんと当てたいですよね。
そこで、今回は戦略（どの順番でチョコレートを食べるか）がより正答率を上げれるかを考えてみました。
## 作戦1：1から順番に食べる
まず、最初に思いつくのが、愚直に1から順番に食べていく方法です。
8つのチョコレートを食べた後、利きチョコを食べることになります。
プログラミング風っぽく書くと以下のように。
（今回のチョコレートは基本的にカカオ含有割合が異なるので、味=カカオ含有量（整数値）で比較して良いでしょう。）
```Cpp
#define N 8

int chocolateCacaoRatioStorage[N+1];   // チョコレートのカカオ含有量を記録する配列
int chocolateCacaoRatio;               // 食べたチョコレートのカカオ含有量
int todayChocolateCacaoRatio;          // 利きチョコレートのカカオ含有量

// 1から8までのチョコレートを食べ、そのカカオ含有量を記録する
for (int i = 1; i <= N; i++) {
    std:cin >> chocolateCacaoRatio;
    chocolateCacaoRatioStorage[i] = chocolateCacaoRatio;
}

// 利きチョコを食べる
std::cin >> todayChocolateCacaoRatio;

// 利きチョコが何番目のチョコレートかを調べる
for (int i = 1; i <= N; i++) {
    if (chocolateCacaoRatioStorage[i] == todayChocolateCacaoRatio) {
        std::cout << "today's chocolate is " << i << std::endl;
        break;
    }
}
```
ではこのプログラムを評価していきましょう。
まず、時間計算量はfor文を2回回しているので、$O(N^2)$です。
ただ、ここに関しては格納方法をハッシュ値にするなどの工夫で$O(N)$に落とすことができます。
一方で問題なのは、空間計算量で、こちらも$O(N)$です。
と、いうことは、8つのチョコレートの味をずっと覚えていかないといけませんね。
筆者の頭は絶対メモリ不足を起こして、利きチョコを食べるころには、おそらく1番目や2番目のチョコレートの味を忘れてしまうでしょう。
もうちょっと確実な方法を考えてみましょう。

## 作戦2：先に利きチョコを食べる
では、先に利きチョコを食べて、その後に8つのチョコレートを食べる方法はどうでしょうか？
```Cpp
#define N 8

int todayChocolateCacaoRatio;  // 利きチョコレートのカカオ含有量を記録する変数
int chocolateCacaoRatio;       // チョコレートのカカオ含有量

// 利きチョコを食べる
std::cin >> todayChocolateCacaoRatio;

// 1から8までのチョコレートを食べ、そのカカオ含有量を利きチョコと比較する
for (int i = 1; i <= N; i++) {
    std:cin >> chocolateCacaoRatio;
    if (chocolateCacaoRatio == todayChocolateCacaoRatio) {
        std::cout << "today's chocolate is " << i << std::endl;
        break;
    }
}
```
今度は、時間計算量は$O(N)$、空間計算量は$O(1)$となりました。
for文を回している間も覚えているべき味は利きチョコの味ただ一つなので、先ほどよりは楽ですね。
とは言え、最悪のケースでは9つのチョコレートを食べることになるので、味が混じったり、忘れたりしないか、やはり少し不安です。

## 作戦3：二分探索
さて、ここまで考えてきた2つの作戦は、どちらも最悪のケースでは9つのチョコレートを食べることになります。
そこで、食べるチョコレートの数を減らす方法を考えてみましょう。
今回のチョコレートは、カカオ含有量が異なり昇順で表されているので、二分探索が使えそうです。
```Cpp
#define N 8

int todayChocolateCacaoRatio;  // 利きチョコレートのカカオ含有量
int chocolateCacaoRatio;       // チョコレートのカカオ含有量

// 利きチョコを食べる
std::cin >> todayChocolateCacaoRatio;

// 二分探索でチョコを食べていく
int min = 1;
int max = N;
int targetChocolate;  // 食べるチョコレートの番号
while (min < max) {
    targetChocolate = (min + max) / 2;
    // targetChocolateを食べる
    std:cin >> chocolateCacaoRatio;
    if (chocolateCacaoRatio == todayChocolateCacaoRatio) {
        std::cout << "today's chocolate is " << i << std::endl;
        break;
    } else if (chocolateCacaoRatio < todayChocolateCacaoRatio) {
        max = targetChocolate;
    } else {
        min = targetChocolate;
    }
}
```
今度は、時間計算量は$O(\log N)$、空間計算量は$O(1)$となりました。
利きチョコの味だけを覚えていればいいですし、最悪のケースでも5つのチョコレートを食べればわかります。
これなら、かなりいけるのでは……？

# 作戦を実行しよう！！

# 最後に
