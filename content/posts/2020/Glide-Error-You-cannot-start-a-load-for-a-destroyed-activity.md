---
title: Glide Error - You cannot start a load for a destroyed activity
date: 2020-05-14T13:45:11+09:00
weight: 1
aliases: []
tags: ["android"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: false
---

Glide 관련 된 처음보는 오류가 발생했다.

`You cannot start a load for a destroyed activity`

다행히(?) 예전에 이슈가 된 에러였다.([https://github.com/bumptech/glide/issues/803](https://github.com/bumptech/glide/issues/803))

이런 에러가 발생한 이유는 `with()` 함수가 lifecycle을 따르기 때문이다. 즉 Glide가 이미지가 로드 중 `Glide.with()`의 `with()`에 들어간 인자가
"activity 또는 fragment가 destroyed"되면서 영향을 받게 되고 위의 에러가 발생한 것으로 보입니다.

[스택오버플로우](https://stackoverflow.com/questions/31964737/glide-image-loading-with-application-context/32887693#32887693)에서 제안하는 방법으로는

아래와 같이 `RequestManager`를 변수에 담아주고 사용하는 것이다.

```java
class MyAdapter extends WhichEveryOneYouUse {
    private final RequestManager glide;
    MyAdapter(RequestManager glide, ...) {
        this.glide = glide;
        ...
    }
    void getView/onBindViewHolder(... int position) {
        // ... holder magic, and get current item for position
        glide.load... or even loadImage(glide, item.url, holder.image);
    }
}
```

```java
loadImage(Glide.with(this), url, findViewById(R.id.image));
// or
list.setAdapter(new MyAdapter(Glide.with(this), data));
```

---

**Reference**

- [https://stackoverflow.com/questions/31964737/glide-image-loading-with-application-context/32887693#32887693](https://stackoverflow.com/questions/31964737/glide-image-loading-with-application-context/32887693#32887693)

- [https://eso0609.tistory.com/72](https://eso0609.tistory.com/72)

- [https://github.com/bumptech/glide/issues/803](https://github.com/bumptech/glide/issues/803)
