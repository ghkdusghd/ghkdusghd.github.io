---
title: "[Github blog] 깃허브 블로그 포스팅 방법"
excerpt: "마크다운 문법을 이용해 md파일을 작성하여 Github blog에 포스팅 해보자."

writer: Dayeong Kim
categories:
  - Github Blog
tags:
  - [Blog, jekyll, Github, Git, markdown]

toc: true
toc_sticky: true

date: 2022-04-28
last_modified_at: 2022-04-28
---

Github Blog에 포스팅하는 방법을 까먹지 않고자 해당 게시글을 작성하고자 한다. 해당 포스트에서는 `markdown`을 사용할 것인데 이를 이용하기 위해선 markdown을 사용할 수 있는 editor가 필요하다. 필자는 포스팅을 위해 `Visual Studio Code`를 이용했다.

에디터를 실행했다면 다음과 같은 과정으로 포스팅을 시작해보자!

# 1. \_posts 폴더 생성

자신이 만든 Github Blog의 로컬 폴더에 `_post` 폴더가 있을 것이다. 이 폴더에 들어가자. Github Blog의 로컬 폴더는 `Github ID.github.io`와 같은 이름을 가지고 있을 것이며, 필자의 Github ID는 DAY0522이므로 필자의 로컬 폴더명은 **DAY0522.github.io**이다.

만약 `_post` 폴더가 존재하지 않으면 새로 생성해주자.

# 2. \_post 폴더에 "yyyy-mm-dd-title.md" 파일을 생성

각각의 게시글을 담당할 `.md` 파일을 작성할 차례다. 각각의 게시글은 **.md**와 같은 확장자를 가지고 있으며, 각 게시글의 파일명은 **yyyy-mm-dd-title**과 같이 작성돼야 한다. title에 작성된 내용은 게시글의 URL에 들어가게 된다.

_ex) 2022-04-28-Github-blog-post.md_

# 3. 머리말을 작성하자.

앞에서 생성한 .md 파일을 열어 머릿말을 작성할 차례이다. Jekyll은 YAML 형식을 이용하여 머리말을 작성한다. 다음의 코드를 통해 머리말을 작성할 수 있으며, 해당 코드는 YAML 형식의 극히 일부이므로 더 많은 내용을 알고 싶다면 [여기](https://jekyllrb-ko.github.io/docs/front-matter/) 또는 구글링을 통해 확인하도록 하자.

```YAML
---
title: "[Github blog] 깃허브 블로그 포스팅 방법"
excerpt: "마크다운 문법을 이용해 md파일을 작성하여 Github blog에 포스팅 해보자."

writer: Dayeong Kim
categories:
  - Gibhub Blog
tags:
  - [Blog, jekyll, Github, Git, markdown]

toc: true
toc_sticky: true

date: 2022-04-28
last_modified_at: 2022-04-28
---
```

# 4. Markdown 문법으로 본문을 작성

머릿말을 만들었으므로 본문을 작성할 차례이다. 본문은 **Markdown** 문법에 기초하여 작성자 입맛대로 코드를 작성해나가면 된다.
코드만 봐서는 원하는대로 코드가 잘 작성됐는지 확인하는 것이 어려우므로 `로컬 서버`를 실행하여 작성한 Markdown이 잘 동작하는지 확인하도록 하자.

# 5. Git에 Push하기

.md 파일을 모두 작성하고 로컬서버에서 확인했는데도 문제가 없다면 이제 Github Blog에 올릴 차례다. 작성한 **.md** 파일을 **Git Push** 하여 원격(Github)에 올려야 한다. 원격 서버에 파일을 올린 후에 조금 기다리면(올라가기까지 어느 정도의 시간이 소요됨) 자신의 Github blog에 포스팅된 것을 확인할 수 있다.

해당 포스팅에서 작성한 .md파일의 코드를 참고용으로 공유하고자 한다. 아래 링크에 들어가 Raw로 열어보면 포스팅을 위해 작성한 머리말, 본문 등 코드를 확인할 수 있다.

[_2022-04-28-Github-blog-post.md_](https://github.com/DAY0522/DAY0522.github.io/blob/master/_posts/2022-04-28-Github-blog-post.md)

---

### 참고 사이트

[Markdown 문법 참고](https://gist.github.com/ihoneymon/652be052a0727ad59601)

[Github Blog 포스팅 방법](https://ansohxxn.github.io/blog/posting/)
