---
layout: post
title: JS Hoisting, let, const
tags: [Javascript, Hoisting, let, const]
---

## JS Hoisting

- 변수 및 함수 선언이 컴파일 단계에서 메모리에 저장되는 현상으로 실제 코드가 상단으로 옮겨지는 것이 아니다.
- var 변수/함수의 선언과 함수선언문이 먼저 메모리에 저장되며, 할당은 코드가 작성된 시점에 실행된다.

```javascript
foo(); // 함수선언문은 호이스팅 되므로 이 구문이 실행전에 메모리에 로드되어 정상실행된다.
foo2(); // var foo2 선언만 호이스팅 되어 할당이 되지 않은 상태에서 실행되어 에러가 발생한다.

function foo() { // 함수선언문 --> 호이스팅 대상
	console.log("hello");
}

var foo2 = function() { // 함수표현식 --> 변수 선언만 호이스팅 대상
  console.log("hello2");
}
```

- var 변수/함수 선언이 함수선언문보다 먼저 호이스팅된다.(var 변수/함수의 할당은 이후에 된다.)

```javascript
var myName = "Heee"; // 값 할당 
var yourName; // 값 할당 X

function myName() { // 같은 이름의 함수 선언
    console.log("myName Function");
}
function yourName() { // 같은 이름의 함수 선언
    console.log("yourName Function");
}

console.log(typeof myName); // > "string"
console.log(typeof yourName); // > "function"
```

```javascript
// 위 코드의 실행순서
var myName; // var 선언 호이스팅(undefined로 초기화된다.)
var yourName;

function myName() { // 같은 이름의 함수 선언
    console.log("myName Function");
}
function yourName() { // 같은 이름의 함수 선언
    console.log("yourName Function");
}

yourName = "Heee"; // 값 할당(function -> string)

console.log(typeof myName); // > "string"
console.log(typeof yourName); // > "function"
```

- let/const 변수선언과 함수표현식에서는 호이스팅이 발생하지 않는다.

## 참고
[https://gmlwjd9405.github.io/2019/04/22/javascript-hoisting.html](https://gmlwjd9405.github.io/2019/04/22/javascript-hoisting.html)
[https://developer.mozilla.org/ko/docs/Glossary/Hoisting](https://developer.mozilla.org/ko/docs/Glossary/Hoisting)

## let, const
- var 사용은 호이스팅이 일어나므로 예상하지 못한 동작을 보일 수 있다. 또한 function-level-scope로 함수가 아닌 곳에서 선언하는 변수는 글로벌 변수로 취급된다.

```javascript
{
    var globalV = 1; // 전역변수
}
console.log(globalV); // 1

for(var globlaI=0; globlaI<3; globlaI++){ // 전역변수
    ;
}
console.log(globlaI); // 3

function localF(){
    var localV = 1; // 지역변수
}
console.log(localV); // Uncaught ReferenceError: localV is not defined
```

- let, const 는 block-level-scope 이므로 위 var 예제의 global 변수들이 모두 지역변수로 선언된다.
- let, const 로 선언된 변수/함수는 호이스팅되지 않는다.
- let, const 는 중복 선언이 불가능하다.(var는 가능)
- let 은 값 재할당이 가능하며, const는 재할당이 불가능하다.

## 참고
[https://happycording.tistory.com/entry/let-const-란-왜-써야만-하는가-ES6](https://happycording.tistory.com/entry/let-const-%EB%9E%80-%EC%99%9C-%EC%8D%A8%EC%95%BC%EB%A7%8C-%ED%95%98%EB%8A%94%EA%B0%80-ES6)