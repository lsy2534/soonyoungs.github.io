---
layout: post
title: Welcome to Blog
date: 2023-03-18 00:00 +0800
last_modified_at: 2023-03-18 00:08:25 +0800
tags: [jekyll theme, jekyll, tutorial]
toc:  true
---
**Git hub를 이용한 블로그 만들기**!
{: .message }

오늘은 토요일 집에 앉아서 git hub를 이용한 블로그를 만들어본다.
생각보다 많은 시간을 소비했다. 방법이야 인터넷 구글 많은 곳에 있지만 방문하는 곳마다 에러가 발생해서 문제를 해결하는 방법을 찾는데에 좀 걸린거 같다. 잘 알려지지 않은 에러들이 너무 많아 블로그를 통해 나처럼 고통받는 사람들에게 환경적인 문제를 어떻게 해결할지에 대한 내생각을 적고자 만들었다.
하나하나 써가며 함 블로그를 사용해볼생각이다.


<a href="https://iingang.github.io/posts/windows-github-set/">[Github 깃허브] Github Blog 생성하기<a>  

위링크를 참고해서 GitHub블로그를 만들었다. 다른 참고자료도 잘 봤지만, 다른 참고자료는 GitHub에서 제공하는 가이드문서를 참고해 만든거같다. 가이드문서에 보면 특별히 jekyll를 설치하는데에 있어서 버전명시를 찾지 못해서 많은 시간을 보냈다. 

## Jekyll

>제킬(Jekyll)은 정적 웹사이트 생성기(Static Site Generator)입니다. 제킬은 마크다운(Markdown)이나 HTML과 같은 소스 파일을 이용하여 정적인 HTML 웹사이트를 생성할 수 있습니다. 제킬은 루비(Ruby)로 개발되었으며, 깃허브(GitHub)를 비롯한 여러 웹사이트에서 호스팅을 지원하고 있습니다.    
제킬을 사용하면 개발자는 HTML, CSS, 자바스크립트 등의 웹 개발 기술에 익숙하지 않아도 마크다운이나 일반 텍스트 파일을 이용하여 웹사이트를 생성할 수 있습니다. 또한, 제킬은 블로그나 개인 웹사이트를 만들기에 적합한데, 테마(Theme) 기능을 제공하여 다양한 디자인의 웹사이트를 쉽게 생성할 수 있습니다.


jeKyll를 설치하려면 우선 Ruby를 설치해야하지만 가장 좋은 방법은 먼저 내가 고른 테마를 Github에 빌드를 하고, 빌드 내용을 확인 후 정상으로 화면이 배포가 된다면, 빌드가 실행되는 루비 버전을 로컬에 설치하는 방법이 가장 좋은 방법이라고 생각된다. 다른 많은 참고 사이트에서는 먼저 로컬에 테마를 옮겨 빌드가 되는지를 확인한 후에 Github에 적용하는 방법을 소개했지만, 나같은 경우 에러가 너무 많이나 역으로 빌드하는 방법을 소개하려한다.

## 맘에드는 내 테마 고르기

<a href="http://jekyllthemes.org/">http://jekyllthemes.org/</a>  
나는 가장 마음에는 고양이 사진이있어서 "Not Pure Poole"이라는 테마를 골랐다.
 
![screenshot.png](/img/20230318/selectthema.png)

"Hompage"라는 버튼 을 선택해서 "Not Pure Poole"이라는 테마 소스들을 확인 할수가 있다. README.md 파일에는 사용방법과 설치방법이 나와있지만 아직은 Ruby를 설치하거나(gem명령어를 사용하려면 Ruby를 설치해야한다.) _config.yml파일을 수정하는 방법은 어려워 택하지 않았다.

![screenshot.png](/img/20230318/fork.png)
우측 맨 상단에 Fork버튼을 클릭해서 내 레파지토리에 저장한다.

![screenshot.png](/img/20230318/fork2.png)
Fork로 이름을 지을때에는 "username".github.io 이렇게 입력합니다.


![screenshot.png](/img/20230318/jellkey.png)
내 레파지토리로 옮겨진 졌다면, 레파지토리 상단에 <b>Setting > Pages > Branch > main or master</b> 를 선택하고 Save버튼을 클릭해 저장해 줍니다.

## Ruby Installer

><a href="https://rubyinstaller.org/downloads/">Ruby Installer</a>

Github에서 빌드 및 실행이되는 Ruby버전은 2.4 ~ 2.7 버전에서 빌드가 된다. 루비를 설치해 로컬에서부터 실행하다보니 이게 왜안되는지 찾기보다 dependency 늪에 빠져 나오기 힘들었다.


Cum sociis natoque penatibus et magnis <a href="#">dis parturient montes</a>, nascetur ridiculus mus. *Aenean eu leo quam.* Pellentesque ornare sem lacinia quam venenatis vestibulum. Sed posuere consectetur est at lobortis. Cras mattis consectetur purus sit amet fermentum.

> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.

Etiam porta **sem malesuada magna** mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.

## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- <mark>To highlight</mark>, use `<mark>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark Otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Footnotes

Footnotes are supported as part of the Markdown syntax. Here's one in action. Clicking this number[^fn-sample_footnote] will lead you to a footnote. The syntax looks like:

{% highlight text %}
Clicking this number[^fn-sample_footnote]
{% endhighlight %}

Each footnote needs the `^fn-` prefix and a unique ID to be referenced for the footnoted content. The syntax for that list looks something like this:

{% highlight text %}
[^fn-sample_footnote]: Handy! Now click the return link to go back.
{% endhighlight %}

You can place the footnoted content wherever you like. Markdown parsers should properly place it at the bottom of the post.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Code

Inline code is available with the `<code>` element. Snippets of multiple lines of code are supported through Rouge. Longer lines will automatically scroll horizontally when needed. You may also use code fencing (triple backticks) for rendering code.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

You may also optionally show code snippets with line numbers. Add `linenos` to the Rouge tags.

{% highlight js linenos %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

### Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

- Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
- Donec id elit non mi porta gravida at eget metus.
- Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

<dl>
  <dt>HyperText Markup Language (HTML)</dt>
  <dd>The language used to describe and define the content of a Web page</dd>

  <dt>Cascading Style Sheets (CSS)</dt>
  <dd>Used to describe the appearance of Web content</dd>

  <dt>JavaScript (JS)</dt>
  <dd>The programming language used to build advanced Web sites and applications</dd>
</dl>

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

### Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](http://placehold.it/800x400 "Large example image")
![placeholder](http://placehold.it/400x200 "Medium example image")
![placeholder](http://placehold.it/200x200 "Small example image")

Align to the center by adding `class="align-center"`:

![placeholder](http://placehold.it/400x200 "Medium example image"){: .align-center}

### Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

-----

Want to see something else added? <a href="https://github.com/vszhub/not-pure-poole/issues/new">Open an issue.</a>

[^fn-sample_footnote]: Handy! Now click the return link to go back.
