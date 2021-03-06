---
layout: post
title: Angular <strong>RxJS - Cold observable과 Hot observable</strong>
subtitle: Cold observable과 Hot observable
categories: angular
section: angular
seq: 13
subseq: 17
description: 옵저버블은 구독(subscribe)되기 전까지 실행되지 않는다고 하였다. 이러한 특성을 갖는 옵저버블을 Cold observable이라 하며 RxJS의 옵저버블은 기본적으로 Cold observable이다. Cold observable은 구독되기 이전에는 데이터 스트림을 방출(emit)하지 않으며 Cold observable을 구독하는 옵저버는 옵저버블이 방출(emit)하는 모든 데이터 스트림을 빠짐없이 처음부터 받을 수 있다.
---

* TOC
{:toc}

![angular Logo](/img/angular-logo.png)

# 1. Cold observable

옵저버블은 구독(subscribe)되기 전까지 동작하지 않는다고 하였다. 이러한 특성을 갖는 옵저버블을 Cold observable이라 한다. RxJS의 옵저버블은 기본적으로 Cold observable이다. Cold observable은 구독되기 이전에는 데이터 스트림을 방출(emit)하지 않으며 **Cold observable을 옵저버가 구독하면 처음부터 동작하기 시작한다.** 따라서 옵저버는 옵저버블이 방출하는 모든 데이터 스트림을 빠짐없이 처음부터 받을 수 있다.

옵저버블을 구독하는 옵저버는 하나 이상일 수 있는데 Cold observable을 구독하는 모든 옵저버들은 구독하는 시점과 상관없이 Cold observable이 방출하는 모든 데이터를 처음부터 빠짐없이 모두 받을 수 있다. 이것은 Cold observable을 구독하는 모든 옵저버는 자신만을 위해 독립적으로 실행하는 옵저버블을 갖게 된다고 볼 수 있는데 이러한 특징을 가리켜 [유니캐스트(unicast)](https://ko.wikipedia.org/wiki/유니캐스트)라 한다.

아래는 Cold observable의 예제이다.

```typescript
import { Observable } from 'rxjs';

/*
구독 함수(subscription function)를 제공하여 Cold observable을 생성한다.
Cold observable의 구독 함수는 옵저버가 옵저버블을 구독(subscribe)할 때
호출되며 구독되기 전까지는 호출되지 않는다.
*/
const numbers$ = Observable.create(observer => {
  console.log('[New subscription created]');

  let i = 1;
  const interval = setInterval(
    /*
    1s마다 숫자를 방출하거나 데이터 스트림의 종료를 알린다.
    Observer 객체의 next 메소드는 옵저버블을 구독한 옵저버에게 데이터를 방출한다.
    Observer 객체의 complete 메소드는 옵저버블을 구독한 옵저버에게 옵저버블 스트림의
    종료를 알린다.
    */
    () => i <= 5 ? observer.next(i++) : observer.complete(),
    1000
  );

  // 구독 취소 시에 호출되는 클린업 함수를 반환한다.(option)
  return () => clearInterval(interval);
});

/*
옵저버가 옵저버블을 구독(subscribe)하면 옵저버블의 구독 함수가 호출된다.
즉, 옵저버블이 처음부터 동작하기 시작한다.
*/
numbers$.subscribe(
  value => console.log(`1st subscription next: ${value}`),  //next
  error => console.log(`1st subscription error: ${error}`), // error
  () => console.log('1st subscription complete')            // complete
);

/*
이미 complete된 옵저버블을 다시 구독하여도 옵저버블의 구독 함수가 호출된다.
즉, 옵저버블이 처음부터 동작하기 시작한다.
*/
setTimeout(() => numbers$.subscribe(
  value => console.log(`2nd subscription next: ${value}`),  //next
  error => console.log(`2nd subscription error: ${error}`), // error
  () => console.log('2nd subscription complete')            // complete
), 6000);
```

<iframe src="https://stackblitz.com/edit/cold-observable-exam?ctl=1&embed=1&file=index.ts&hideNavigation=1" frameborder="0" width="100%" height="600"></iframe>

# 2. Hot observable

Hot observable은 옵저버블을 생성하자마자 구독과 상관없이 바로 데이터 스트림을 방출(emit)하기 시작한다. 따라서 일정 시간이 경과한 시점에 옵저버블을 구독하면 데이터 스트림의 앞부분은 구독할 수 없고 중간 부분부터 구독하게 된다. Hot observable은 구독 시점부터 방출되는 데이터를 받는 것을 기본으로 한다.

RxJS의 옵저버블은 기본적으로 Cold observable이다. Cold observable을 Hot observable로 만들려면 publish, share 오퍼레이터를 사용할 수도 있으나 `Subject`를 사용하면 편리하다. Subject는 옵저버블이면서 옵저버이다. 따라서 옵저버블을 구독할 수도 있으며 옵저버처럼 next, complete 메소드를 직접 호출할 수 있다. 이처럼 next 메소드를 호출할 수 있으므로 Subject를 사용하면 데이터를 방출할 수도 있다. 이것은 Cold observable의 유니캐스트한 특성과는 달리 Hot observable을 구독하고 있는 모든 옵저버에게 부수 효과(side-effect)가 있다는 의미이다. 이러한 특징을 가리켜 [멀티캐스트(multicast)](https://ko.wikipedia.org/wiki/멀티캐스트)라 한다.

아래는 Hot observable의 예제이다.

```typescript
import { Subject, BehaviorSubject } from 'rxjs';

const subject = new Subject();
const numbersBySubject$ = subject.asObservable();

// 데이터 방출
subject.next(1);
subject.next(2);
subject.next(3);

// 이 시점에는 구독할 데이터가 없다.

numbersBySubject$.subscribe(
  value => console.log(`1st next: ${value}`),  //next
  error => console.log(`1st error: ${error}`), // error
  () => console.log('1st complete')            // complete
);

numbersBySubject$.subscribe(
  value => console.log(`2nd next: ${value}`),  //next
  error => console.log(`2nd error: ${error}`), // error
  () => console.log('2nd complete')            // complete
);

// 데이터 방출
subject.next(10);
subject.next(20);
subject.next(30);

/*----------------------------------------------------*/
// BehaviorSubject는 Observable의 마지막 데이터만을 받는다.
const behaviorSubject = new BehaviorSubject<number>(0);
const numbersByBehaviorSubject$ = behaviorSubject.asObservable();

// 마지막 데이터만을 받는다.
behaviorSubject.next(100);
behaviorSubject.next(200);
behaviorSubject.next(300);

numbersByBehaviorSubject$.subscribe(
  value => console.log(`1st next: ${value}`),  //next
  error => console.log(`1st error: ${error}`), // error
  () => console.log('1st complete')            // complete
);
```

<iframe src="https://stackblitz.com/edit/subject-exam?ctl=1&embed=1&file=index.ts&hideNavigation=1" frameborder="0" width="100%" height="600"></iframe>

# 3. 유니캐스트 vs 멀티캐스트

Cold observable은 유니캐스트하고 Hot observable은 멀티캐스트하다.

Cold observable을 구독하는 모든 옵저버는 자신만을 위해 독립적으로 실행하는 옵저버블을 갖게 된다. 다시 말해 옵저버블과 옵저버는 일대일(one-to-one)의 관계를 갖는다. 하지만 Hot observable를 구독하고 있는 모든 옵저버에게 부수 효과(side-effect)가 있다. 다시 말해 옵저버블과 옵저버는 일대다(one-to-many)의 관계를 갖는다.

![](/img/unicast-multicast.png)

유니캐스트 vs 멀티캐스트
{: .desc-img}

아래는 유니캐스트와 멀티캐스트의 예제이다.

```typescript
import { Observable, Subject } from 'rxjs';

/* Unicast */
const coldObservable$ = Observable.create(
  // 랜덤 데이터 방출
  observer => observer.next(Math.random())
);

/* Cold observable을 구독하는 모든 옵저버는 자신만을 위해 독립적으로 실행하는 옵저버블을 갖게 된다. */
coldObservable$.subscribe(
  value => console.log(`1st Cold observable's observer: ${value}`)
);

coldObservable$.subscribe(
  value => console.log(`2nd Cold observable's observer: ${value}`)
);

/* Multicast */
const subject = new Subject();
const hotObservable$ = subject.asObservable();

/* 구독하고 있는 모든 옵저버에게 부수 효과(side-effect)가 있다. */
hotObservable$.subscribe(
  value => console.log(`1st Hot observable's observer: ${value}`)
);

hotObservable$.subscribe(
  value => console.log(`2nd Hot observable's observer: ${value}`)
);

// 랜덤 데이터 방출
subject.next(Math.random());
```

<iframe src="https://stackblitz.com/edit/unicast-multicast?ctl=1&embed=1&file=index.ts&hideNavigation=1" frameborder="0" width="100%" height="600"></iframe>

