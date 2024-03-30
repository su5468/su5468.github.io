---
layout: posts
categories:
    - Development
title: css box-sizing 0
---

# Intro

간단한 html 문서를 만들 일이 있어서 만들던 중, 검색창을 구현할 일이 있었다.  
`height`를 같게 해주었음에도 `input` 요소와 `button` 요소의 높이가 다른 크기를 가지는 것을 볼 수 있었는데, 처음엔 여백 문제일까 싶었지만, `margin`이나 `padding` 등에도 문제가 없음을 파악했다.  
CSS에는 같은 크기로 명시된 두 요소의 크기가 달랐던 것인데, 이는 바로 `box-sizing` 때문이었다.  

# CSS Box Model

html 요소의 크기에는 무엇이 관여하는가?  
요소 그 자체의 크기(높이와 너비), Padding(안쪽 여백), Border(경계선), 그리고 Margin(바깥쪽 여백)이다.  
이를 CSS Box Model이라 하는데, 사실 여기까지만 생각하면 저 값들이 전부 같은 요소들은 크기가 같으리라고 추측할 수 있다.  
그러나, 실제로는 그렇지 않은데, 왜냐하면 요소에 따라 크기를 정하는 방식이 다를 수 있기 때문이다.  

# box-sizing

크기를 정하는 방식이라, 말이 조금 어렵게 느껴지니 실제 예시를 생각해 보자.  

```css
button {
    width: 100px;
    padding: 10px;
    border: 10px solid;
}

button.content {
    box-sizing: content-box;
}

button.border {
    box-sizing: border-box;
}
```

위와 같은 두 개의 버튼이 있다고 했을 때, 두 버튼의 가로 크기는 같을 것 같지만, 실제로는 `box-sizing`의 차이로 인해 다른 크기를 지니게 된다.  
먼저, `content` 버튼은 요소 그 자체의 너비가 `100px`이며, 양 옆으로 `padding`과 `border`가 더해지므로 전체 너비는 `140px`가 될 것이다.  
한편, `border` 버튼은 `padding`과 `border`가 모두 더해진 전체 너비가 `100px`이고, 요소 그 자체의 원래 너비는 `60px`로 쪼그라들 것이다.  

즉, `box-sizing: content-box`인 요소는 `padding`과 `border`를 제외한 높이와 너비를 설정할 때 사용하고, `box-sizing: border-box`인 요소는 `padding`과 `border`를 더했을 때의 높이와 너비를 설정할 때 사용한다.  

# box-sizing의 기본값

참고로, MDN에 따르면 요소별 `box-sizing`의 기본값은 아래와 같다.

- **`content-box`**: 일반적인 모든 요소들의 기본값.
- **`border-box`**: `<table>`, `<select>`, `<button>` 및 `radio`, `checkbox`, `reset`, `button`, `submit`, `color`, `search` 타입의 `<input>` 요소의 기본값.

# 결론

html 요소의 크기 및 여백이 예상과 다른 경우, `box-sizing`을 확인하는 것이 해답이 될 수 있다.  
`box-sizing: content-box`인 경우 요소의 크기는 순수하게 요소 자체의 크기만을 의미한다.  
`box-sizing: border-box`인 경우 요소의 크기는 `padding`과 `border`를 포함한 값이다.  
```css
box-sizing : content-box | border-box ;
```

# 참조

- [스택오버플로우 - Button size is different that's in styles](https://stackoverflow.com/questions/12105811/button-size-is-different-thats-in-styles)
- [MDN - Introduction to the CSS basic box model](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model)
- [MDN - box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)