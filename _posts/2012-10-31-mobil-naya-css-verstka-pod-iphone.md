---
title: Мобильная CSS-верстка под iPhone
author: mike
layout: post
permalink: /2012/mobil-naya-css-verstka-pod-iphone/
Hide SexyBookmarks:
  - 0
Hide OgTags:
  - 0
categories:
  - Кодинг
tags:
  - css
  - safari mobile
---
Нужно было создать решение для страницы, которая открывается как в мобильных браузерах, так и в десктопных. Писать сложное решение с использованием серверного кода не хотелось, а вставлять костыли типа <a href="http://www.conditional-css.com/" target="_blank">Conditional CSS</a>  &#8211; тоже некрасивая и нетривиальная задача.

Однако, как и ожидалось, решение было найдено и оказалось весьма простым.

<!--more-->Способы условного подключения css-файлов для страницы можно найти в интернете.В данном же конкретном случае разбирается то, что Safari Mobile не хочет представляться как handheld-устройство (видимо считая себя выше этого).

Решение  этой проблемы таково: знание размеров мобильного устройства и соотношение его сторон.

Так например, можно использовать такой код:

<pre class="theme:classic-spaced lang:css decode:true">&lt;meta name="viewport" content="minimum-scale=0.5, maximum-scale=0.5, width=device-width, user-scalable=no"&gt;
&lt;link rel="stylesheet" type="text/css" href="style-portrait.css" media="screen and (max-width: &lt;cutoff&gt;px)"&gt;
&lt;link rel="stylesheet" type="text/css" href="style-landscape.css" media="screen and (min-width: &lt;cutoff+1&gt;px)"&gt;</pre>

чтобы разделить портретную и пейзажную ориентации. Главное, не забывать корректно проставлять параметр cutoff. Cutoff означает среднее число между длиной и короткой стороной устройства. Так, например, iPhone имеет размеры 320х480 и iPad &#8211; 768х1024. Соответственно, значения cutoff для них будут: 400 и 896 (заменять <var><cutoff></var> и<var><cutoff+1></var> в примере нужно соответственно на 400 и на 401 для iPhone)

Если вы предпочитаете подгружать код используя @import (от этой привычки я избавился, прочитав <a href="http://www.stevesouders.com/blog/2009/04/09/dont-use-import/" target="_blank">don&#8217;t use import</a>), то можно использовать следующий вариант:

<pre class="theme:classic-spaced lang:default decode:true">&lt;style type="text/css"&gt;
/* iPad */
@media screen and (min-device-width: 768px) {
    @media screen and (max-width: 896px) { @import url('ipad-portrait.css'); }
    @media screen and (min-width: 897px) { @import url('ipad-landscape.css'); }
}
/* iPhone */
@media screen and (max-device-width: 480px) {
    @media screen and (max-width: 400px) { @import url('iphone-portrait.css'); }
    @media screen and (min-width: 401px) { @import url('iphone-landscape.css'); }
}
&lt;/style&gt;</pre>

Приведённые примеры показывают, как один и тот же сайт можно, не изменяя контента, привести к виду, одинаково удобоваримому на всех устройствах.

Примеры взяты с <a href="http://library.13bold.com/developing-themes-for-bowtie/tasty-recipes/" target="_blank">13bold.com</a>.

P.S.: Кстати, [вот здесь][1] можно посмотреть кто как читает подключаемые CSS.

 [1]: http://www.alistapart.com/articles/return-of-the-mobile-stylesheet