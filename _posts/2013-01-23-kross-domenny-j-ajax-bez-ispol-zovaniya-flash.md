---
title: Кросс-доменный AJAX без использования Flash
author: mike
layout: post
categories:
  - Кодинг
tags:
  - ajax
  - cross-domain ajax
  - javascript
  - кроссдоменный ajax
---
CORS &#8211; существующая на протяжении некоторого времени технология создания JS запросов к другому домену. Однако, ранее я с ней не встречался и гуглинг по рунету и по хабру, в частности, не привёл к ощутимым результатам. Поэтому сейчас я предлагаю вам перевод [статьи ][1] с Mozilla Developers Network, где использование этой технологии достаточно хорошо описывается.

# Контроль доступа по HTTP (с использованием CORS)

Кросс-доменные HTTP запросы &#8211; это HTTP запросы на ресурсы из другого домена, не совпадающего с доменом ресурса, делающего запрос. Например, ресурс загруженный по адресу А (<a href="http://domaina.example)" class="autohyperlink" title="http://domaina.example)" target="_blank">domaina.example)</a>,  например HTML страница, делает запрос на ресурс, находящийся в домене B (<a href="http://domainb.foo)" class="autohyperlink" title="http://domainb.foo)" target="_blank">domainb.foo)</a>, например такой, как изображение, используя при этом тэг IMG  (http://domainb.foo/image.jpg). Сегодня в интернете это обычная практика &#8211; страницы загружают огромное количество ресурсов с других сайтов, в том числе CSS стили, изображеня, скрипты и другие ресурсы.

Однако, кросс-доменный HTTP запрос, инициированый внутри javascript-кода, имел известные ограничения по вполне понятным причинам безопасности. Например, HTTP-запросов с использованием объекта XMLHttpRequest подвергался требованиям [Same-origin policy][2]. В частности, это означает, что веб-приложение, использующее XMLHttpRequest может сделать HTTP-запросы только к домену, с которого оно было загружено, а не в другого домен. Поэтому разработчики выразили желание развить возможности  XMLHttpRequest для межсайтовых запросов, для лучшей, более безопасной работы с веб-приложениями.  
<!--more-->

  
[Web Applications Working Group][3] в рамках  [W3C][4] предложила новую  [Cross-Origin Resource Sharing][5] (CORS) рекомендацию, которая предоставляет возможность веб-серверам, поддерживать безопасный способ кросс-доменного запроса. Особо следует отметить, что данная спецификация используется в API контейнерах, таких как XMLHttpRequest, как смягчающий механизм, позволяющий расширить механизм ограничения &#8220;одного и того же домена&#8221; в современных браузерах. Информация, представленная в данной статье, представляет интерес для веб-администраторов, разработчиков серверов и веб-разработчиков. Другая статья, предназначенная для серверных программистов и [обсуждющая реализацию CROS с их точки зрения (с примерами на PHP)][6], содержит дополнительную информацию. Находясь на клиентской стороне, современный браузер обрабатывает компоненты крос-доменного запроса, в том числе заголовки и политики. Однако, введение этой новой возможности, вовсе не означает, что серверы должны обрабатывать новые заголовки запроса и возвращается правильные ответы.

Это стандарт кросс-доменного обмена используется для включения межсайтового запроса в следующих случаях:

  * кросс-доменные вызовы XMLHttpRequest, так как это описано выше.
  * Web Fonts (для загрузки шрифтов, используя @font face в CSS), так что серверы могут использовать TrueType шрифты, которые могут быть только загружены и использованы только веб-сайтами, которым разрешено это делать.
  * WebGL текстур.
  * Изображения, помещённые на canvas с помощью DrawImage.

Эта статья представляет собой общее обсуждение Cross-Origin Resource Sharing, и включает в себя обсуждение HTTP заголовки как это реализовано в Firefox 3.5.

## Обзор

Стандарт CORS  работает за счет добавления новых заголовков HTTP, позволяющих серверам описать адреса, которым разрешено читать эту информацию, используя веб-браузер. Кроме того, для методов HTTP-запроса, которые могут оказывать влияние на пользовательские данные (в частности, для HTTP методы, отличные от GET, или для методов POST  с определенными типами MIME), спецификация обязывает браузеры &#8220;предвыполнить&#8221; запрос, определяя, поддерживаются ли эти методы с помощью заголовка HTTP OPTIONS, а затем, после &#8220;одобрения&#8221; от сервера, отправить фактический запрос с реальными данными. Серверы могут также уведомлять клиентов том, что раличного рода &#8220;credentials&#8221; (в том числе и Cookies и  HTTP Authentication data) должны быть отправлены вместе с запросами.

Последующие разделы обсуждают сценарии, а также разбивку заголовков HTTP используется.

## Примеры сценариев с контролем доступа

Здесь мы представляем три сценария, которые являются иллюстрацией того, как Cross-Origin Resource Sharing работает. Все эти примеры относятся к использованию XMLHttpRequest, который может быть использован для межсайтового вызова в любом поддерживающем его браузере.

Наличие фрагментов JavaScript, включенных в эти разделы (и экземпляров серверного кода, который корректно обрабатывает эти межсайтовые запросоы) можно проверить в действии [здесь][7]. Страница будет работать в браузерах, которые поддерживают межсайтового XMLHttpRequest. Обсуждение Cross-Origin Resource Sharing с точки зрения сервера (в том числе примеры PHP кода) можно найти [здесь][6].

## Простые запросы

Простой кросс-доменный запрос, это запрос, который:

  * Только использует GET или POST. Если POST используется для отправки данных на сервер, Content-Type данных, отправляемых на сервер с запросом HTTP POST является одним из следующего списка: application/х-WWW-form-urlencoded, multipart/form-data или text/plain.
  * Не установлены пользовательские заголовки HTTP-запроса (например, X-Modified и т.д.)

> Примечание: Это те же виды межсайтовых запросов, что веб-контент может уже вопрос, и ответ не данные выпущен на запрос, если сервер посылает соответствующий заголовок. Таким образом, сайты, предотвратить перекрестное сайте подделке запросов ничего нового не опасаться контроля доступа HTTP.

Например, предположим, что страница на домене <a href="http://foo.example" class="autohyperlink" title="http://foo.example" target="_blank">foo.example</a> намерена получить содержимое с домена <a href="http://bar.other" class="autohyperlink" title="http://bar.other" target="_blank">bar.other</a>. Код такого рода может быть использован с помощью яваскрипта, расположенного на foo.example:

{% highlight javascript %}
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/public-data/';

function callOtherDomain() {
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
}
{% endhighlight %}


Вот что отсылает и получает браузер:

<pre class="lang:default decode:true">GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: <a href="http://foo.example/examples/access-control/simpleXSInvocation.html" class="autohyperlink" title="http://foo.example/examples/access-control/simpleXSInvocation.html" target="_blank">foo.example/examples/access-control/simpleXSInvocation.html</a>
Origin: <a href="http://foo.example" class="autohyperlink" title="http://foo.example" target="_blank">foo.example</a>

HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61 
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[XML Data]</pre>

Строки 1 &#8211; 10 являются заголовками, отправляемыми Firefox 3.5. Обратите внимание, что основным заголовком запрос в данном примере является Origin: заголовок в строке 10 и выше, который показывает, что вызов происходит от содержимого в домене <a href="http://foo.example" class="autohyperlink" title="http://foo.example" target="_blank">foo.example</a>.

Строки 13 &#8211; 22 показывают HTTP ответа от сервера в домене <a href="http://bar.other" class="autohyperlink" title="http://bar.other" target="_blank">bar.other</a>. В ответ сервер посылает обратно заголовок Access-Control-Allow-Origin: как показано в строке 16. Использование заголовков Origin: и Access-Control-Allow-Origin: показывает, что протокол управления доступом достаточно просто для использования. В нашем случае сервер отвечает Access-Control-Allow-Origin: *. Это означает, что его ресурсы могут быть доступны для любого домена при помощи кросс-сайтового запроса. Если бы владелец ресура <a href="http://bar.other" class="autohyperlink" title="http://bar.other" target="_blank">bar.other</a> хотел, ограничить доступ к ресурсу, так чтобы он мог быть использован только с <a href="http://foo.example" class="autohyperlink" title="http://foo.example" target="_blank">foo.example</a>, то он мог бы отправить обратно:

<pre class="lang:default decode:true">Access-Control-Allow-Origin: <a href="http://foo.example" class="autohyperlink" title="http://foo.example" target="_blank">foo.example</a></pre>

Обратите внимание, что сейчас ни один домен, кроме <a href="http://foo.example" class="autohyperlink" title="http://foo.example" target="_blank">foo.example</a> (у которых Origin: заголовок в запросе, указанный в строке 10 выше), не может получить доступ к ресурсу с помощью кросс-доменного запроса. Access-Control-Allow-Origin заголовок должен содержать значение, которое было отправлено в Origin: заголовке запроса.

Продолжение читайте в следующей статье.

 [1]: https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS
 [2]: https://developer.mozilla.org/en-US/docs/JavaScript/Same_origin_policy_for_JavaScript
 [3]: http://www.w3.org/2008/webapps/ "http://www.w3.org/2008/webapps/"
 [4]: http://www.w3.org/ "http://www.w3.org/"
 [5]: http://www.w3.org/TR/cors/ "http://www.w3.org/TR/cors/"
 [6]: https://developer.mozilla.org/En/Server-Side_Access_Control
 [7]: http://arunranga.com/examples/access-control/