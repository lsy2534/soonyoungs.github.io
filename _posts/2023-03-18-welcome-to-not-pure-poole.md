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

