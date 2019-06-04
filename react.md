# OPGG React/JSX Style Guide

[Airbnb React/JSX Style Guide
](https://github.com/airbnb/javascript/tree/master/react)([🇰🇷한국어](https://github.com/apple77y/javascript/tree/master/react))를 기반으로 오피지지 프론트엔드팀 코드리뷰에서 나온 내용을 기준으로 한다.

## 시스템 기준

- React v16.8.6(2019.06.04)기준

## 정렬

[Prettier](https://prettier.io/)를 이용하여 정렬, 따옴표, 띄어쓰기 등 코드스타일이 달라지는 문제를 쉽게 해결한다.

- [Prettier Options](https://prettier.io/docs/en/options.html)의 기본 옵션을 사용한다.
- VSCode의 경우 확장프로그램[Prettier - Code formatter
  ](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)을 사용한다.
  > Format On Save 옵션을 켜고 사용하는 것을 권장한다.
- intelliJ도 마찬가지로 [Prettier - Plugins | JetBrains](https://plugins.jetbrains.com/plugin/10456-prettier)을 사용한다. (VSCode와 다르게 기본 설정파일 추가를 해줘야한다.)

## 네이밍

- 확장자: 컴포넌트 파일 `.jsx` 그 외 자바스크립트 파일 `.js`
- 변수: 카멜케이스 [Camel case - Wikipedia]
- 컴포넌트: 파스칼케이스
  > PascalCase for upper camel case (after the Pascal programming language) - [Camel case - Wikipedia]

## 폴더구조

- components: 컴포넌트
- hooks: 리액트 훅 - [Introducing Hooks
  ](https://reactjs.org/docs/hooks-intro.html)
- lib: JS 라이브러리
- docs: 문서(주로 마크다운)
- i18n: 다국어
- styles: style관련 파일들(color, constants, GlobalStyle, mixin 등)
- (Redux를 사용하는 경우)Ducks Pattern 사용 - [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux)

## 참고

- [Airbnb React/JSX Style Guide
  ](https://github.com/airbnb/javascript/tree/master/react)

<!-- 변수 -->

[camel case - wikipedia]: https://en.wikipedia.org/wiki/Camel_case
