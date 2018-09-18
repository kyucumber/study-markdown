# JavaScript에서 DOM 활용하기

JavaScript에서의 DOM 객체에 대해 알아보자.

## Node Interface

DOM Node Interface의 IDL(interface description language)이 DOM Level 3 Core spec정의되어 있다. [DOM Level 3 Core Spec](https://www.w3.org/TR/DOM-Level-3-Core/core.html#ID-1950641247)

솔직히 영어라서 잘 모르겠고, 직접 Node 를 출력해보자

```javascript
for(var key in Node){
    console.log(key + ' = ' + Node[key]);
};
console.dir(Node)
```

자바스크립트로 직접 값을 찍어보면 아래처럼 상수로 정의된 Node의 값들이 출력된다.

Node는 EventTarget을 \_\_proto\_\_로 가지며, EventTarget를 상속하는 구조임을 알 수 있다.

![](/images/js/node.png)

각각의 노드들은 아래와 같은 상속 구조를 가진다. HTMLElement 하위에는 저 5개 뿐만 아니라, 엄청 많은 Element들이 더 있다.

![](/images/js/domNode.png)

평소에 div나, button이나 그런 DOM들에 어떻게 전부 addEventListener가 가능했을까?

아무 버튼이나 querySelctor든, getElementById든 가져와 보면 아래처럼 \_\_proto\_\_ 속성이 HTMLButtonElement로 되어 있을 것이다.

![](/images/js/button.png)

 HTMLButtonElement는 EventTarget의 자식이고, 여기서 프로토타입 체이닝을 통해서 EventTarget의 prototype에 있는 addEventListener 메소드를 사용할 수 있다.

EventTarget도 console.dir로 찍어보면, prototype에 addEventListener메소드가 들어있다.

## DOM Navigation

특정 속성들에 의해 Node들은 연결되어 있다. 간단한 html과 js를 통해 Node들을 출력해보자

example.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1>first node</h1>
    <ul>
        <li>first li</li>
        <li>second li</li>
    </ul>
</body>
</html>
```

example.js

```javascript
console.log(document.documentElement); //<html>
console.log(document.body); //<body>
console.log(document.body.children[0]); //<h1>
console.log(document.body.firstElementChild); //<h1>
```

위에서 유의할 점은, childNodes 메소드는 Node를 가져오기 때문에 아래처럼 text, element 구분 없이 전부 가져오게 된다. 

![](/images/js/childNodes.png)

`<h1>`, `<ul>`로 이루어진 body에 5개나 들어가 있는데,이는 `<h1>`앞의 공백 또한 text로 Node로 취급하기 때문이다. 그에 반해 `children` 메소드는 Element들만을 출력하기 때문에 아래처럼 h1과 ul만 출력된다.

![](/images/js/children.png)

아마 Node를 쓰기 보다는, Element를 활용하는 경우가 많을테니 children을 쓰면 된다.

## Node와 Element 메소드

아마 아래의 메소드들은 유용하게 쓰일거 같다. parent는 부모 DOM, previousSibling은 이전 형제, child는 자식, first, last는 각각 첫번째 마지막 자식 DOM을 가져온다.

![](/images/js/nodeAndElement.png)

위 Element 밑에 childNodes는 오타, children을 해야 자식 Element들만 가져온다.﻿

## DOM Search

javascript 코드 안에서 DOM을 검색하려면 아래처럼 id, class, tag, name 프로퍼티를 이용해 가져올 수 있다. querySelector를 쓰는게 제일 편한 듯 싶다.

```javascript
document.getElementById('id');
//단일 개체 반환
document.getElementByName('name');
//같은 name 속성값이 존재할 수 있으므로 배열로 반환
document.getElementByTagName('tagName');
//element 내부에 메소드로 존재, 배열 반환
document.getElementsByClassName('className');
//element 내부 메소드 존재, 배열 반환
document.querySelector('id, class, ..')
//element 내부 메소드 존재, 단일 개체 반환
document.querySelectorAll('id, classes');
//element 내부 메소드 존재, 배열 반환
```

## DOM Attribute

DOM attribute 와 property 둘 다 한글 번역으로는 속성으로, 처음엔 같다고 생각했고 지금도 헷갈리지만 두개는 약간 다른 개념이다. 간단하게 말하면 DOM properties는 자바스크립트 객체의 속성, HTML attributes는는 HTML 태그의 속성이다.

example.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1 id="firsth">first node</h1>
    <ul>
        <li>first li</li>
        <li>second li</li>
    </ul>
    <input id="input1" type="text" value="이거값이안바뀔거다아마">
</body>
</html>
```

example.js

```javascript
var h1 = document.querySelector('#firsth');
console.log(h1.hasAttribute('id')); //true
// attribute => property
h1.setAttribute('id', 'child-id');
console.log("property: ", h1.id);

// property => attribute
h1.id = 'id-2';
console.log("attribute: ", h1.getAttribute('id'));

var valueAttr = input1.getAttribute('value');
console.log("attribute: ", valueAttr);
console.log("property: ", input1.value);

input1.value = 'changeValue'; // 사용자가 인풋박스에 입력을 해도 마찬가지 이다.
console.log("attribute: ", input1.getAttribute('value')); // 변경이 없다.
```

위에서 h1이 가진 HTML의 id는 firsth(attribute)였고, setAttribute로 attribute를 변경했을 때 property에도 적용이 되는 것을 알 수 있다. 서로 변경 시 대응되는 상태이다.

근데, 아래의 input 태그의 경우에는 property를 바꾸면 input 박스의 값은 바뀌지만, 실제로 HTML DOM이 가지는 attribute의 값은 변경되지 않는다. **input 태그의 value 속성은 property와 attribute값이 id처럼 대응되지 않는다.** 

## DOM 변경(생성, 삽입, 복사, 삭제)하기

- 요소 생성

```javascript
var el = document.createElement('h1'); //parameter tagName
aa.innerHTML = 'hello this is h1';

var txtNode = document.createTextNode('hello'); //parameter value
```

- 요소 삽입

```javascript
//element.appendChild(Node); 해당 엘리먼트의 자식으로 노드 추가
//element.append(Element); 해당 엘리먼트의 자식으로 Element 추가

var newLi = document.createElement('li');
newLi.innerHTML = 'hello';

document.body.insertBefore(newLi, document.body.childNodes[0]);
//제일 처음에 newLi 추가

document.querySelector('ul').replaceChild(newLi, document.querySelector('ul').children[0]);
//첫번째 li child를 newLi로 변경

document.querySelector('#firsth').insertAdjacentHTML('afterend', '<h1>마지막에 들어가</h1>');
//element 뒤에 추가하기

// element.insertAdjacentHTML(position, text);
//'beforebegin' element 앞에 
//'afterbegin' element 안에 가장 첫번째 child
//'beforeend' element 안에 가장 마지막 child
//'afterend' element 뒤에

```

replaceChild의 경우 eventListener를 제거하기 위해 replaceChild로 child를 날리거나 할 때도 사용한다.

- 요소 복제

```javascript
var dupNode = node.cloneNode(deep);
//node 복제되어야 할 node.
//dupNode 복제된 새로운 node.
//deep Optional 해당 node의 children 까지 복제하려면 true, 해당 node 만 복제하려면 false
```

- 요소 삭제

```javascript
parentElement.removeChild(node);
node.remove();
```