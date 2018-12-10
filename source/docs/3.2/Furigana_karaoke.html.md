---
title: 注音卡拉OK
---
{::options toc_levels="1..6" /}

[[img/Furigana-demo-1.png]]

_Furigana_ (in Aegisub often shortened to _furi_) refers to little phonetic
guide characters written along the main text in Japanese, specifically using
the hiragana phonetic alphabet to describe how the ideographic kanji characters
should be pronounced. Putting smaller text next to a main line of text is in
general referred to as [_ruby
text_](http://en.wikipedia.org/wiki/Ruby_character), but since the
implementation discussed here is designed specifically with Japanese furigana
in mind, the ruby text is also referred to as furigana everywhere.

Furigana（在Aegisub中经常缩写为furi）指的是沿日文主文本上写的小的注音字符，特别是使用平假名来描述表意的汉字字符如何发音。将较小的文本放在主文本行旁边通常称为ruby text(旁注文字)，但由于此处讨论的实现是专门为日文假名注音设计的，因此ruby text在任何地方也被称为furigana(假名注音)。

None of the subtitle formats Aegisub supports natively support ruby text or
furigana, however the [[karaskel|Automation/Lua/Modules/karaskel.lua]] standard include
implements an algorithm that can create basic furigana layouts by calculating
the position of every individual character.

没有任何字幕格式被Aegisub本地支持ruby text或furigana，但[[karaskel | Automation / Lua / Modules / karaskel.lua]]标准包括实现一种可以通过计算每个字符的位置来创建基本的furigana布局的算法。

This page describes the syntax the Automation 4 karaskel.lua script understands
for furigana text, and how to use the layout information it calculates to
actually create positioned characters.

此页面描述了Automation 4 karaskel.lua脚本理解furigana文本的语法，以及如何使用它计算的布局信息来实际创建定位字符。

[[Karaoke Templater|Automation/Karaoke_Templater]] also implements support for
furigana using the karaskel.lua algorithm and syntax.

Karaoke Templater也通过使用karaskel.lua的算法和语法实现了对furigana的支持。

It's important to note that the syntax is designed for karaoke, and revolves
around [[karaoke timed|Karaoke_timing]] text. It isn't suited for typesetting
regular text (e.g. dialogue lines) with general purpose ruby text. A more
elaborate syntax and more complex layout engine would be required for that.

重要的是要注意语法是为卡拉OK设计的，并以卡拉OK定时文本为核心。它不适合用于用通用的ruby text去排版常规文本（例如对话行）。为此，需要更精细的语法和更复杂的布局工具。

## Multi-highlight syntax  ##
A prerequisite for an integral part of the furigana syntax is the
multi-highlight syntax.

If you make the text of a syllable a number sign (#, ASCII 35, Unicode U+0023)
that syllable will "join" with the previous one: The number sign is removed and
the timing of the two syllables are added together, producing just one
syllable. You can have multiple number sign syllables in a row, adding up
multiple timings in that way.

多重高亮语法
furigana语法不可或缺的一个先决条件是多重高亮语法。
如果你将一个音节的文本设为数字符号（＃，ASCII 35，Unicode U + 0023），则该音节将与前一个音符“连接”：删除数字符号即将两个音节的时间相加，合成为一个音节。在一行中您可以使用多个数字符号音节，以这种方式添加多个计时。

The timings of the individual number sign syllables are still stored in the
[[highlight table|Automation/Lua/Modules/karaskel.lua#highlighttable]] of the
generated syllable structure, but the main timing (`start_time` and `end_time`)
of the syllable structure reflects only the added-together timings of the
number sign syllables.

各个数字符号音节的计时信息仍存储在所生成的音节结构的高亮表中，但是音节结构的主计时（start_time和end_time）仅反映数字符号音节的相加计时。

{::template name="examplebox"}
This line shows how multi-highlight syntax is used to mark up kanji and groups
of kanji that cover multiple syllables:

例子
此行展示如何使用多重高亮语法标记涵盖多个音节的汉字和汉字组：


    {\k5}明日{\k10}#{\k5}#{\k10}ま{\k7}た{\k10}会{\k4}う{\k6}時{\k14}#

It generates the following syllable structures:

其生成了以下音节结构：

<table class="karatable">
    <tr><th>Text</th><th>Syllable duration</th><th>Highlight durations</th></tr>
    <tr><td rowspan="3">明日</td><td rowspan="3">20</td><td>5</td></tr>
    <tr><td>10</td></tr>
    <tr><td>5</td></tr>
    <tr><td>ま</td><td>10</td><td>10</td></tr>
    <tr><td>た</td><td>7</td><td>7</td></tr>
    <tr><td>会</td><td>10</td><td>10</td></tr>
    <tr><td>う</td><td>4</td><td>4</td></tr>
    <tr><td rowspan="2">時</td><td rowspan="2">20</td><td>6</td></tr>
    <tr><td>14</td></tr>
</table>

{:/}

## Basic furigana  ##
To add furigana to a syllable, you add a pipe character (|, ASCII 124, Unicode
U+007C) after the main syllable text, and then add the furigana text after the
pipe. You can also add furigana to repeat-syllables (number sign syllables for
multi-highlight) to have the furigana for a single main syllable span multiple
furigana syllables.

When multiple consecutive syllables all have furigana, the furigana for all of
those syllables are collected together and centered above the string of main
syllables they belong to. If the string of furigana is wider than the main text
the furigana is left-aligned with the main text. You can control this behaviour
with special control characters, see below.

基础furigana
要给一个音节添加furigana，请在主音节文本后添加管道符（|，ASCII 124，Unicode U + 007C），然后在管道符后添加假名注音文本。您也可以为重复音节（用于多重高亮的数字符号音节）添加假名注音，以使单个主音节由多个假名注音。
当多个连续的音节都有假名注音时，所有这些音节的假名都被收集在一起，并且位于它们所属的主音节字符串的正上方。如果注音假名的字符串比主文本宽，则假名字符与主文本左对齐。您可以使用特殊控制字符控制此行为，请参阅下文。

{::template name="examplebox"}
Adding furigana to the example above:

在上面的示例中添加注音假名：


    {\k5}明日|あ{\k10}#|し{\k5}#|た{\k10}ま{\k7}た{\k10}会|あ{\k4}う{\k6}時|と{\k14}#|き

The following syllables, highlights and furigana are produced:

生成了以下音节，高亮和注音假名：

<table class="karatable">
    <tr><th>Text</th><th>Syllable duration</th><th>Highlight/furigana durations</th><th>Furigana</th></tr>
    <tr><td rowspan="3">明日</td><td rowspan="3">20</td><td>5</td><td>あ</td></tr>
    <tr><td>10</td><td>し</td></tr>
    <tr><td>5</td><td>た</td></tr>
    <tr><td>ま</td><td>10</td><td>10</td></tr>
    <tr><td>た</td><td>7</td><td>7</td></tr>
    <tr><td>会</td><td>10</td><td>10</td><td>あ</td></tr>
    <tr><td>う</td><td>4</td><td>4</td></tr>
    <tr><td rowspan="2">時</td><td rowspan="2">20</td><td>6</td><td>と</td></tr>
    <tr><td>14</td><td>き</td></tr>
</table>
{:/}

## Controlling the layout  ##
Often the layout produced with the plain furigana syntax isn't exactly what you
want, or maybe even plain misleading. Because of this, there's two special
characters that can be used to control how the furigana are laid out.

Both of these two special characters are placed before the first character of
the furigana of a syllable, i.e. right after the pipe character.

First is the exclamation mark (!, ASCII 33, Unicode U+0021) which marks a
"sequence break". This acts as a kind of invisible divider that prevents the
furigana in this syllable from merging with that of the previous syllable. You
will usually use this when you have two adjacent kanji words that both have
furigana, but the furigana for them need to be separate. In that case, put the
exclamation mark as the first character in the furigana for the first syllable
of the second word.

The other special character is the less-than sign (<, ASCII 60, Unicode U+003C)
which marks a "sequence break with float-left". It has the same sequence break
semantics as the exclamation mark, but also changes the overflow behaviour.
When the furigana sequence starts with a less-than sign marked furigana
syllable is wider than the main text it applies to, it will always center above
the main text, even if it means it has to extend over the left edge of it.

In all cases, if two furigana sequences extend beyond their main text such that
they would overlap, the main text is moved such that the furigana won't
overlap.

控制布局
通常使用简单的假名注音语法生成的布局并不完全是您想要的，或者甚至可能是误导性的。因此，有两个特殊字符可用于控制注音假名的布局方式。
这两个特殊字符都放在一个音节的第一个注音假名字符之前，即紧跟在管道字符之后。
第一个字符是用于标记“序列中断”的感叹号（！，ASCII 33，Unicode U + 0021）。这是一种不可见的分隔符，可以防止这个音节中的注音假名与前一个音节的注音假名重合。当你有两个相邻的汉字都有注音假名时，你通常会使用它，但是它们的假名必须是分开的。在这种情况下，将感叹号作为第二个汉字的第一个音节的注音假名的第一个字符。
另一个特殊字符是小于号（<，ASCII 60，Unicode U + 003C），它标志着“浮动左侧的序列中断”。它具有与感叹号相同的序列中断语义，但改变了溢出行为。当注音假名序列以小于号的方式开始时，被标记的注音假名音节比它所应用的主文本宽，它将位于主文本的正上方(上方居中)，即使这意味着它必须延伸到它的左边缘以外。
在所有情况下，如果两个注音假名序列延伸超出其主文本以使它们重叠，则移动主文本使得注音假名不会重叠。



{::template name="examplebox"}
Here is the same (rather contrived) sample text shown without layout control
and with each of the two layout control characters:

这是相同的（相当人为的）示例文本，演示了没有布局控制符和两个布局控制符中的每一个：

| [[img/Furigana-demo-4.png]] | `{\k10}`中\|ちゅ`{\k10}`#\|う`{\k10}`国\|ご`{\k10}`#\|く<br>`{\k10}`<u>魂\|た</u>`{\k10}`#\|ま`{\k10}`#\|し`{\k10}`#\|い
| [[img/Furigana-demo-3.png]] | `{\k10}`中\|ちゅ`{\k10}`#\|う`{\k10}`国\|ご`{\k10}`#\|く<br>`{\k10}`<u>魂\|!た</u>`{\k10}`#\|ま`{\k10}`#\|し`{\k10}`#\|い
| [[img/Furigana-demo-2.png]] | `{\k10}`中\|ちゅ`{\k10}`#\|う`{\k10}`国\|ご`{\k10}`#\|く<br>`{\k10}`<u>魂\|&lt;た</u>`{\k10}`#\|ま`{\k10}`#\|し`{\k10}`#\|い

It _is_ very hard to tell the difference between the two first as the
difference is only a few pixels, but it is there. In the first sample, the た
extends a bit over the left edge of 魂 and above 国 while it exactly
left-aligns with 魂 in the second. In the second, ちゅうごく is also centered
above 中国 while it isn't in the first.
{:/}

由于差异仅为几个像素，因此很难区分两者之间的差异，但差异是存在的。在第一个例子中，た略微延伸，超出了魂的左边界到了国的上方，而在第二个例子中它与魂完全左对齐。并且与第一个例子不同的是，ちゅうごく也位于中国的正上方(上方居中)。


## Summary  ##

|------|-------|------------------|--------------------------------|----------
| Char | ASCII | Unicode          | Where                          | Meaning
|:----:|:-----:|------------------|--------------------------------|----------
| \#   | 35    | U+0023<br>U+FF03 | Instead of main text           | Extend previous syllable with another highlight
| \|   | 124   | U+007C<br>U+FF5C | Between main text and furigana | Separate main text and furigana text of a syllable
| !    | 33    | U+0021<br>U+FF01 | First character of furigana    | Sequence break; prevent joining furigana for this syllable with furigana from previous syllable
| &lt; | 60    | U+003C<br>U+FF1C | First character of furigana    | Sequence break with float-left; prevent joining furigana for this syllable with furigana from previous syllable, but allow furigana to extend left of main text
|------|-------|------------------|--------------------------------|----------
{:.karatable}

Note that every special character can in fact be represented by two different
Unicode codepoints. The first is the regular character, corresponding to the
ASCII character, while the second (high) codepoint is the _full width_ version
of the character. Often when using an IME (Input Method Editor) to edit
Japanese text it is easier to input text in full width mode than switching the
IME off to enter a single or two regular ASCII characters and switch it on
again. Therefore both the half width (ASCII) and full width versions of the
characters are accepted.

## Usage in Karaoke Templater  ##
Furigana: [[The _furi_ template class|Automation/Karaoke_Templater/Template_modifiers#furi]]

Multi-highlight: [[The _multi_ modifier|Automation/Karaoke_Templater/Template_modifiers#multi]]

{::template name="examplebox"}
The examples used earlier on this page are all generated using this kara-templater snippet:

    Comment: 0,0:00:00.00,0:00:00.00,Default,,0000,0000,0000,template syl,{\pos(!line.left+syl.center!,!line.middle!)\an5\k!syl.start_time/10!\k$kdur}
    Comment: 0,0:00:00.00,0:00:00.00,Default,,0000,0000,0000,template furi,{\pos(!line.left+syl.center!,!line.middle-line.height!)\an5\k!syl.start_time/10!\k$kdur}
    Comment: 0,0:00:00.00,0:00:02.00,Default,,0000,0000,0000,karaoke,{\k15}二|ふ{\k15}#|た{\k10}人|り{\k15}だ{\k57}け{\k5}の{\k6}地|ほ{\k5}球|し{\k8}で
    Comment: 0,0:00:02.00,0:00:04.00,Default,,0000,0000,0000,karaoke,{\k10}中|ちゅ{\k10}#|う{\k10}国|ご{\k10}#|く{\k10}魂|<た{\k10}#|ま{\k10}#|し{\k10}#|い
    Comment: 0,0:00:04.00,0:00:06.00,Default,,0000,0000,0000,karaoke,{\k10}中|ちゅ{\k10}#|う{\k10}国|ご{\k10}#|く{\k10}魂|!た{\k10}#|ま{\k10}#|し{\k10}#|い
    Comment: 0,0:00:06.00,0:00:08.00,Default,,0000,0000,0000,karaoke,{\k10}中|ちゅ{\k10}#|う{\k10}国|ご{\k10}#|く{\k10}魂|た{\k10}#|ま{\k10}#|し{\k10}#|い

The font used in MS PMincho 30 pt with the furigana being 15 pt.
{:/}


## Usage in Lua scripts  ##
It's all in [[karaskel|Automation/Lua/Modules/karaskel.lua]].

Furigana layout is automatically invoked by `karaskel.preproc_line_pos` if a
furigana style exists for a line main style. The furigana style for a main
style is a style with the same name, except `-furigana` appended to the name.
E.g. the furigana style of `Default` is `Default-furigana`.

Karaskel can generate automatic furigana styles if the `generate_furigana`
argument (second) to the `karaskel.collect_head` function is `true`. Automatic
furigana styles are identical to the main style they're based on, except the
font size is halved.

Furigana syllables are stored in `line.furi` and follows the same format as
regular syllables. You have to remember setting the style of the lines you
generate to the furigana style.

Multi-highlights are always processed even when furigana layout isn't done.
Multi-highlight data are stored in `syl.highlights`.

{::template name="todo"}more details{:/}

{::template name="automation_navbox" /}
