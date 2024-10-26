---
title: "Intl.PluralRules"
date: 2024-10-21T12:57:49Z
author: "skyfe79"
draft: false
tags: ["javascript","web","아마추어 번역","V8","\bECMAScript"]
---

국제화는 쉽지 않다. 복수형 처리는 언뜻 간단해 보이지만, 실제로 모든 언어마다 고유한 복수형 규칙이 있어 복잡하다.

영어의 복수형은 두 가지 경우만 있다. "cat"이라는 단어를 예로 들어보자:

- 1 cat, 즉 `'one'` 형태로 영어에서는 단수형이라고 한다.
- 2 cats, 42 cats, 0.5 cats 등, 즉 `'other'` 형태(유일한 다른 형태)로 영어에서는 복수형이라고 한다.

새로운 [`Intl.PluralRules` API](https://github.com/tc39/proposal-intl-plural-rules)는 주어진 숫자에 따라 선택한 언어에서 어떤 형태를 사용해야 하는지 알려준다.

```javascript
const pr = new Intl.PluralRules('en-US');
pr.select(0);   // 'other' (예: '0 cats')
pr.select(0.5); // 'other' (예: '0.5 cats')
pr.select(1);   // 'one'   (예: '1 cat')
pr.select(1.5); // 'other' (예: '1.5 cats')
pr.select(2);   // 'other' (예: '2 cats')
```

다른 국제화 API와 달리, `Intl.PluralRules`는 저수준 API로 직접 형식을 지정하지 않는다. 대신 이를 기반으로 자체 형식 지정 함수를 만들 수 있다:

```javascript
const suffixes = new Map([
  // 주의: 실제 상황에서는 복수형을 이렇게 하드코딩하지 않고,
  // 번역 파일의 일부로 포함시킬 것이다.
  ['one',   'cat'],
  ['other', 'cats'],
]);

const pr = new Intl.PluralRules('en-US');
const formatCats = (n) => {
  const rule = pr.select(n);
  const suffix = suffixes.get(rule);
  return `${n} ${suffix}`;
};

formatCats(1);   // '1 cat'
formatCats(0);   // '0 cats'
formatCats(0.5); // '0.5 cats'
formatCats(1.5); // '1.5 cats'
formatCats(2);   // '2 cats'
```

비교적 단순한 영어 복수형 규칙에는 이 방법이 과도해 보일 수 있다. 하지만 모든 언어가 같은 규칙을 따르지는 않는다. 일부 언어는 단 하나의 복수형만 있고, 어떤 언어는 여러 형태가 있다. 예를 들어, [웨일스어](http://unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html#rules)는 6가지 복수형이 있다!

```javascript
const suffixes = new Map([
  ['zero',  'cathod'],
  ['one',   'gath'],
  // 주의: 'two' 형태는 이 단어의 경우 'one' 형태와 우연히 같지만,
  // 웨일스어의 모든 단어가 그런 것은 아니다.
  ['two',   'gath'],
  ['few',   'cath'],
  ['many',  'chath'],
  ['other', 'cath'],
]);

const pr = new Intl.PluralRules('cy');
const formatWelshCats = (n) => {
  const rule = pr.select(n);
  const suffix = suffixes.get(rule);
  return `${n} ${suffix}`;
};

formatWelshCats(0);   // '0 cathod'
formatWelshCats(1);   // '1 gath'
formatWelshCats(1.5); // '1.5 cath'
formatWelshCats(2);   // '2 gath'
formatWelshCats(3);   // '3 cath'
formatWelshCats(6);   // '6 chath'
formatWelshCats(42);  // '42 cath'
```

여러 언어를 지원하면서 정확한 복수형을 구현하려면 언어별 복수형 규칙 데이터베이스가 필요하다. [유니코드 CLDR](http://cldr.unicode.org/)에는 이 데이터가 포함되어 있지만, JavaScript에서 사용하려면 다른 JavaScript 코드와 함께 포함시켜 제공해야 한다. 이는 로딩 시간, 파싱 시간, 메모리 사용량을 증가시킨다. `Intl.PluralRules` API는 이 부담을 JavaScript 엔진으로 옮겨, 더 효율적인 국제화된 복수형 처리를 가능하게 한다.

**주의:** CLDR 데이터에는 언어별 형태 매핑이 포함되어 있지만, 개별 단어의 단수/복수형 목록은 포함되어 있지 않다. 이전과 마찬가지로 이는 직접 번역하고 제공해야 한다.

## 서수

`Intl.PluralRules` API는 선택적 `options` 인자의 `type` 속성을 통해 다양한 선택 규칙을 지원한다. 기본값은 `'cardinal'`이다. 대신 주어진 숫자의 서수 표현을 알아내려면 (예: `1` → `1st`, `2` → `2nd` 등) `{ type: 'ordinal' }`을 사용한다:

```javascript
const pr = new Intl.PluralRules('en-US', {
  type: 'ordinal'
});

const suffixes = new Map([
  ['one',   'st'],
  ['two',   'nd'],
  ['few',   'rd'],
  ['other', 'th'],
]);

const formatOrdinals = (n) => {
  const rule = pr.select(n);
  const suffix = suffixes.get(rule);
  return `${n}${suffix}`;
};

formatOrdinals(0);   // '0th'
formatOrdinals(1);   // '1st'
formatOrdinals(2);   // '2nd'
formatOrdinals(3);   // '3rd'
formatOrdinals(4);   // '4th'
formatOrdinals(11);  // '11th'
formatOrdinals(21);  // '21st'
formatOrdinals(42);  // '42nd'
formatOrdinals(103); // '103rd'
```

`Intl.PluralRules`는 다른 국제화 기능에 비해 저수준 API다. 직접 사용하지 않더라도, 이를 사용하는 라이브러리나 프레임워크를 사용할 수 있다.

이 API가 더 널리 사용 가능해지면, [Globalize](https://github.com/globalizejs/globalize#plural-module)와 같은 라이브러리들이 하드코딩된 CLDR 데이터베이스 의존성을 네이티브 기능으로 대체하여 로딩 시간, 파싱 시간, 실행 시간 성능과 메모리 사용량을 개선할 것이다.

## `Intl.PluralRules` 지원 현황

- Chrome: 버전 63부터 지원
- Firefox: 버전 58부터 지원
- Safari: 버전 13부터 지원
- Node.js: 버전 10부터 지원
- Babel: 지원하지 않음

## 알림

이 글은 [v8.dev](https://v8.dev/)에 2017년 10월 4일 발행된 [Intl.PluralRules](https://v8.dev/features/intl-pluralrules) 글을 한국어로 편역한 내용을 담고 있습니다.

