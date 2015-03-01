---
title: A Fancy AJAX Contact Form
author: vladandreev
layout: post
permalink: /2012/a-fancy-ajax-contact-form/
Hide SexyBookmarks:
  - 0
Hide OgTags:
  - 0
dsq_thread_id:
  - 954609853
categories:
  - Кодинг
tags:
  - jQuery plugins
  - обзор
---
<p style="text-align: left;">
  На днях мне довелось поработать с формами на одном из наших сайтов. В качестве руководства я использовал статью <a title="A Fancy AJAX Contact Form" href="http://tutorialzine.com/2009/09/fancy-contact-form/" target="_blank">A Fancy AJAX Contact Form</a>. Автор (<a title="Martin" href="http://www.linkedin.com/in/martinaglv" target="_blank">Martin Angelov</a>) рассказывает как придать формам на сайте приятный и лаконичный вид, настроить js-валидацию и AJAX-обработку данных. Также предлагается посмотреть Demo и скачать готовый пример.
</p>

[<img class="aligncenter size-full wp-image-60" title="A Fancy AJAX Contact Form" src="http://re-coders.com/blog/wp-content/uploads/2012/10/406.jpg" alt="" width="620" height="340" />][1]

<!--more-->

В качестве валидатора используется jquery-plugin <a title="validationEngine" href="http://www.position-absolute.com/articles/jquery-form-validator-because-form-validation-is-a-mess/" target="_blank">validationEngine</a>, а <a title="jqtransform" href="http://www.dfc-e.com/metiers/multimedia/opensource/jqtransform/" target="_blank">jqtransform</a> — для изменения внешнего вида. Что и как подключается достаточно подробно описано <a title="A Fancy AJAX Contact Form" href="http://tutorialzine.com/2009/09/fancy-contact-form/" target="_blank">там же</a>, так что я не буду заострять на этом вашего внимания. Лучше поведаю вам об одной проблеме, которая приключилась со мной в процессе интеграции этого чуда на сайт.

Итак, все замечательно, если вы работаете с формой на странице сайта. Но стоит поместить форму во всплывающее окно, все становится куда интереснее. После открытия такого окна, всплывающие подсказки формы теряются в пространстве &#8211; они прилипают к левому краю экрана, тогда как должны появляться рядом с соответствующими полями.

Из этой ситуации нашелся следующий выход. Отказаться от управления видимостью окна средствами css-свойства display. Вместо этого, оно скрывается от пользователя свойством {top: -100%}, а отображается свойством {top: 20%}. Таким образом, форма с самого начала уже присутствует на странице и всплывающие подсказки больше не теряются! <img src="http://re-coders.com/blog/wp-includes/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />

&nbsp;

UPD: Чтобы решение работало и на небольших экранах, нужно вместо {top: -100%} использовать {top: -1000px}, где 1000px &#8211; это некая величина превышающая высоты всплывающего окна.

 [1]: http://re-coders.com/blog/wp-content/uploads/2012/10/406.jpg