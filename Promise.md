## Promise

<br>

### Promise가 뭘까?

mdn에서 프로미스의 정의를 살펴보자.

> Promise는 프로미스가 생성될 때 꼭 알 수 있지는 않은 값을 위한 대리자로, 비동기 연산이 종료된 이후의 결과값이나 실패 이유를 처리하기 위한 처리기를 연결할 수 있도록 한다. 프로미스를 사용하면 비동기 메서드에서 마치 동기 메서드처럼 값을 반환할 수 있다. 다만 최종 결과를 반환하지는 않고, 대신 프로미스를 반환해서 미래의 어떤 시점에 결과를 제공한다.

정의를 알듯 말듯 하다. 

'처리기를 연결할 수 있도록 한다'의 의미는 프로미스 사용 구문을 살펴보면 알 수 있다.

> new Promise(*excutor*);

위 구문에서 excutor가 처리기를 받는 부분이다. 처리기 부분에는 콜백 함수를 인자로 받는데, 그 콜백 함수의 첫번째 인자는 resolve함수이며, 두번째 인자는 reject함수이다. 이름에도 알 수 있듯 resolve는 비동기 작업이 정상적으로 완료하면 어떤 작업을 수행하도록 도와주는 함수이고, reject함수는 오류가 발생하였을 경우 오류를 처리할 수 있도록 돕는 함수이다.

'최종 결과를 반환하지 않고, 대신 프로미스를 반환'의 의미는 아마 다음과 같은 의미인 것 같다.

프로미스를 반환한다는 의미는 함수에서 프로미스를 반환한다는 의미로 해석할 수 있을 것 같다. 비동기를 처리하기 위해 프로미스를 사용하면 보통 함수 내에서 프로미스를 반환하는 형식으로 사용하게 된다. 코드로 살펴보자.

```javascript
const asyncProcessor = () => {
  return new Promise((resolve, reject) => 'Do Something'); // 프로미스를 반환하고 있다.
}
```

주석 처럼 asyncProcessor 함수는 프로미스를 리턴하고 있다. 'Do Something' 부분에서는 비동기작업을 수행하고, 그 결과를 바탕으로 resolve나 reject를 실행하게 된다.

<br>

### Promise는 상태를 가진다?!

프로미스는 다음과 같이 세 가지 상태를 가진다. 역시 mdn을 살펴보자.

- 대기(*pending*): 이행하거나 거부되지 않은 초기 상태
- 이행(*fulfilled*): 연산이 성공적으로 완료됨 
- 거부(*rejected*): 연산이 실패함

대기 중인 프로미스는 값과 함께 이행할 수도 있고, 오류와 같은 이유로 인해 거부될 수 있다. 이행이나 거부될 때, 프로미스에 연결한 처리기는 그 프로미스의 then 메서드에 의해 대기열에 오른다. 이미 이행하였거나 거부된 프로미스에 연결한 처리기도 호출하므로, 비동기 연산과 처리기 연결 사이에 경합 조건은 없다.

역시나 읽어도 알듯 말듯 하다.

![promises](https://mdn.mozillademos.org/files/8633/promises.png)

그림을 보면서 이해해보자.

가장 왼쪽의 프로미스의 상태는 대기 상태이다. 비동기 작업이 완료되면 결과에 따라 resolve 혹은 reject 함수를 호출하여 이행하거나, 거부하게 될 것이다. ~~결과가 이도저도 아니면?!~~

이행 혹은 거부가 되면 프로미스의 then에서 확인할 수 있다. (then은 이따가 살펴보자.) settled는 이행 또는 거부됐을 때의 상태를 의미하며 ''처리되었다''라는 의미를 갖는다고 한다. 아무튼, then에서는 비동기 작업(?)(actions)을 하거나, 오류를 다루거나 또다른 프로미스를 리턴하게 된다. 오류를 다룰 때에는 then을 사용할 수 있지만, 보통 then 끝에 catch 메서드를 사용하여 한 번에 다루는 경향이 있는 것 같다.

이제 then을 살펴보자.

###Pomise.prototype.then   

then 메서드는 프로미스를 리턴하고, 두 개의 콜백 함수를 인자로 받는다. 하나는 프로미스가 이행했을 때의 함수이며, 다른 하나는 거부했을 때를 위한 콜백 함수이다. 구문으로 살펴보자.

> *p.then(onFulfilled, onRejected)*

프로미스가 이행하거나 거부했을 때 각각의 상태를 담당하는 함수가 *비동기적으로 실행*된다. then에서 프로미스가 거부했을 때도 다루기는 하지만, catch 메서드가 있으므로 이행한 상태만 살펴보자. 이행을 담당하는 핸들러 함수는 다음 규칙을 따라 실행된다.

- 핸들러 함수가 값을 반환한 경우, then에서 반환한 프로미스는 그 반환값을 자신의 결과값으로 하여 이행한다.
- 값을 반환하지 않는 경우, then에서 반호나한 프로미스는 undefined를 결과값으로 하여 이행한다.
- 이미 이행한 프로미스를 반환할 경우, then에서 반환한 프로미스는 그 프로미스의 결과값을 자신의 결과값으로 하여 이행한다.
- 대기 중인 프로미스를 반환할 경우, then에서 반환한 프로미스는 그 프로미스의 이행 여부와 결과값을 따른다.



then과 catch 메서드는 프로미스를 반환하기 때문에, 체이닝이 가능하다. (아마 위의 핸들러 함수의 규칙에 따라서 핸들러 함수가 프로미스를 반환해야 체이닝이 가능한 것 같다.)

여기까지.

