---
title: 'Modern Javascript Deep Dive - 45장 프로미스'
date: 2023-09-19
category: 'Javascript'
draft: false
---

ES5에서는 비동기 처리를 위한 또 다른 패턴으로 프로미스를 도입했다.
프로미스는 전통적인 콜백 패턴이 가진 단점을 보완하며 비동기 처리 시점을 명확하게 표현할 수 있다는 장점이 있다.

# 1. 비동기 처리를 위한 콜백 패턴의 단점
## 1.1 콜백 헬
다음은 GET 요청을 위한 함수다.
```jsx
// GET 요청을 위한 비동기 함수  
const get = url => {  
    const xhr = new XMLHttpRequest();  
    xhr.open('GET', url);  
    xhr.send();  
  
    xhr.onload = () => {  
        if (xhr.status === 200) {  
            // 서버의 응답을 콘솔에 출력한다.  
            console.log(JSON.parse(xhr.response));  
        } else {  
            console.error(`${xhr.status} ${xhr.statusText}`);  
        }  
    };  
};  
  
// id가 1인 post를 취득  
get('https://jsonplaceholder.typicode.com/posts/1');
```

get 함수는 비동기 함수다.
비동기 함수를 호출하면 함수 내부의 비동기로 동작하는 코드가 완료되지 않았다 해도 기다리지 않고 즉시 종료된다.
즉, 비동기 함수 내부의 비동기로 동작하는 코드는 비동기 함수가 종료된 이후에 완료된다.
**-> 비동기 함수 내부의 비동기로 동작하는 코드에서 처리 결과를 외부로 반환하거나 상위 스코프의 변수에 할당하면 기대한 대로 동작하지 않는다.**

setTimeout 함수를 활용한 코드다.
```jsx
let g = 0;  
  
// 비동기 함수인 setTimeout 함수는 콜백 함수의 처리 결과를 
// 외부로 반환하거나 상위 스코프의 변수에 할당하지 못한다.  
setTimeout(() => { g = 100;}, 0);  
console.log(g); // 0
```

onload 이벤트 핸들러가 비동기로 동작한다.
-> get 함수의 onload 이벤트 핸들러에서 서버의 응답 결과를 반환하거나 상위 스코프의 변수에 할당하면 기대한 대로 동작하지 않는다.

```jsx
// GET 요청을 위한 비동기 함수  
const get = url => {  
    const xhr = new XMLHttpRequest();  
    xhr.open('GET', url);  
    xhr.send();  
  
    xhr.onload = () => {  
        if (xhr.status === 200) {  
            // 서버의 응답을 콘솔에 출력한다.    
console.log(JSON.parse(xhr.response));  
        } else {  
            console.error(`${xhr.status} ${xhr.statusText}`);  
        }  
    };  
};  
  
// id가 1인 post를 취득  

const response = get('https://jsonplaceholder.typicode.com/posts/1');  
console.log(response); // undefined
```

xhr.onload 핸들러 프로퍼티에 바인딩한 이벤트 핸들러가 즉시 실행되는 것이 아니다.
xhr.onload 이벤트 핸들러는 load 이벤트가 발생하면 일단 태스크 큐에 저장되어 대기하다가, 콜 스택이 비면 이벤트 루프에 의해 콜 스택으로 푸시되어 실행된다.
</br>
**비동기 함수는 처리 결과를 외부에 반환할 수 없고, 상위 스코프의 변수에 할당할 수도 없다.**
따라서 비동기 함수의 처리 결과(서버의 응답 등)에 대한 후속 처리는 비동기 함수 내부에서 수행해야 한다.
이때 비동기 함수를 범용적으로 사용하기 위해 비동기 함수에 비동기 처리 결과에 대한 후속 처리를 수행하는 콜백 함수를 전달하는 것이 일반적이다.
필요에 따라 처동기 처리가 성공하면 호출될 콜백 함수와 비동기 처리가 실패하면 호출될 콜백 함수를 전달할 수 있다.

```jsx
// GET 요청을 위한 비동기 함수
const get = (url, successCallback, failureCallback) => {  
    const xhr = new XMLHttpRequest();  
    xhr.open('GET', url);  
    xhr.send();  
  
    xhr.onload = () => {  
        if (xhr.status === 200) {  
            // 서버의 응답을 콜백 함수에 인수로 전달하면서 호출하여 응답에 대한 후속처리를 한다.  
            successCallback(JSON.parse(xhr.response));  
        } else {  
            // 에러 정보를 콜백 함수에 인수로 전달하면서 호출하여 에러 처리를 한다.  
            failureCallback(xhr.status);  
        }  
    };  
};  
  
// id가 1인 post를 취득  
// 서버의 응답에 대한 후속 처리를 위한 콜백 함수를 비동기 함수인 get에 전달한다.  
get('https://jsonplaceholder.typicode.com/posts/1', console.log, console.error);
```

이처럼 콜백 함수를 통해 비동기 처리 결과에 대한 후속 처리를 수행하는 비동기 함수가 비동기 처리 결과를 가지고 또다시 비동기 함수를 호출해야 한다면 콜백 함수 호출이 중첩되어 복잡도가 높아지는 현상이 발생하는데,
이를 **콜백 헬** 이라 한다.

```jsx
// GET 요청을 위한 비동기 함수
const get = (url, callback) => {  
    const xhr = new XMLHttpRequest();  
    xhr.open('GET', url);  
    xhr.send();  
  
    xhr.onload = () => {  
        if (xhr.status === 200) {  
            // 서버의 응답을 콜백 함수에 인수로 전달하면서 호출하여 응답에 대한 후속처리를 한다.  
            callback(JSON.parse(xhr.response));  
        } else {  
  
            console.error(`${xhr.status} ${xhr.statusText}`);  
        }  
    };  
};  
  
const url = 'https://jsonplaceholder.typicode.com/posts/1';  
  
// id가 1인 post의 userId를 취득  
get(`{$url}/posts/1`, ({userId}) => {  
    console.log(userId); // 1  
    // post의 userId를 사용하여 user 정보를 취득  
    get(`${url}/users/${userId}`, userInfo => {  
        console.log(userInfo);  
    });  
});
```

GET 요청을 통해 서버로부터 응답을 취득하고 이 데이터를 사용하여 또다시 GET 요청을 한다.
콜백 헬은 가독성을 나쁘게 하며 실수를 유발하는 원인이 된다.
다음이 콜백 헬이 발생하는 전형적인 사례다.
```jsx
get('/step1', a => {  
   get(`/step2/${a}`, b => {  
      get(`/step3/${b}`, c => {  
         get(`/step4/${c}`, d => {  
             console.log(d);   
			});   
		});  
   });  
});
```

## 1.2 에러 처리의 한계
비동기 처리를 위한 콜백 패턴의 문제점 중에서 가장 심각한 것은 에러 처리가 곤란하다는 점이다.

```jsx
try {  
    setTimeout(() => {  
        throw new Error(`Error!`);  
    }, 1000);  
} catch (e) {  
    // 에러를 캐치하지 못한다.  
    console.error('캐치한 에러', e);  
}
```

**에러는 호출자(caller) 방향으로 전파된다.**
즉, 콜 스택의 아래 방향(실행 중인 실행 컨텍스트가 푸시되기 직전에 푸시된 실행 컨텍스트 방향)으로 전파된다.
하지만 setTimeout 함수의 콜백 함수를 호출한 것은 setTimeout 함수가 아니다.
-> setTimeout 함수의 콜백 함수가 발생시킨 에러는 catch 블록에서 캐치되지 않는다.

# 2. 프로미스의 생성

Promise 생성자 함수를 new 연산자와 함께 호출하면 프로미스를 생성한다.
Promise 생성자 함수는 비동기 처리를 수행할 콜백 함수를 인수로 전달받는데 이 콜백 함수는 resolve와 reject 함수를 인수로 전달받는다.

```jsx
// 프로미스 생성  
const promise = new Promise((resolve, reject) => {  
   // Promise 함수의 콜백 함수 내부에서 비동기 처리를 수행한다.  
    if (/* 비동기 처리 성공 */) {  
        resolse('result');  
    } else {  
        // 비동기 처리 실패  
        reject('failure reason');  
    }  
});
```

Promise 생성자 함수가 인수로 전달받은 콜백 함수 내부에서 비동기 처리를 수행한다.
이때 비동기 처리가 성공하면 콜백 함수의 인수로 전달받은 resolve 함수를 호출하고,
비동기 처리가 실패하면 reject 함수를 호출한다.
다음은 비동기 함수 get을 프로미스를 사용하여 구현했다.

```jsx
// GET 요청을 위한 비동기 함수  
const promiseGet = url => {  
    return new Promise((resolve, reject) => {  
        const xhr = new XMLHttpRequest();  
        xhr.open('GET', url);  
        xhr.send();  
  
        xhr.onload = () => {  
            if (xhr.status === 200) {  
                // 성공적으로 응답을 전달받으면 resolve 함수를 호출한다.  
                resolve(JSON.parse(xhr.response));  
            } else {  
                // 에러 처리를 위해 reject 함수를 호출한다.  
                reject(new Error(xhr.status));  
            }  
        };  
    });  
};  
  
promiseGet('https://jsonplaceholder.typicode.com/posts/1');
```

비동기 처리는 Promise 생성자 함수가 인수로 전달받은 콜백 함수 내부에서 수행한다.
프로미스는 다음과 같이 현재 비동기 처리가 어떻게 진행되고 있는지를 나타내는 상태 정보를 갖는다.

| **프로미스의 상태정보** | **의미**                              | **상태 변경 조건**                  |
| ----------------------- | ------------------------------------- | ------------------------------- |
| pending                 | 비동기 처리가 아직 수행하지 않은 상태 | 프로미스가 생성된 직후 기본상태 |
| fulfilled               | 비동기 처리가 수행된 상태(성공)       | resolve 함수 호출               |
| rejected                | 비동기 처리가 수행된 상태(실패)       | reject 함수 호출                |

생성된 직후의 프로미스는 기본적으로 pending 상태다. 이후 비동기 처리가 수행되면 비동기 처리 결과에 따라 다음과 같이 프로미스의 상태가 변경된다.
- 비동기 처리 성공: resolve 함수를 호출해 프로미스를 fulfilled 상태로 변경한다.
- 비동기 처리 실패: reject 함수를 호출해 프로미스를 rejected 상태로 변경한다.

**프로미스의 상태는 resolve 또는 reject 함수를 호출하는 것으로 결정된다.**

fulfilled 또는 rejected 상태를 settled 상태라고 한다.
settled 상태는 fulfilled 또는 rejected 상태를 settled 상태와 상관없이 pending이 아닌 상태로 비동기 처리가 수행된 상태를 말한다.

**-> 프로미스는 비동기 처리 상태와 처리 결과를 관리하는 객체다.**

# 3. 프로미스와 후속 처리 메서드
**프로미스의 비동기 처리 상태가 변화하면 후속 처리 메서드에 인수로 전달한 콜백 함수가 선택적으로 호출된다.**

## 3.1 Promise.prototype.then

then 메서드는 두 개의 콜백 함수를 인수로 전달받는다. (fulfilled, reject)

첫 번째 콜백 함수는 비동기 처리가 성공했을 때 호출되는 성공 처리 콜백 함수이며,
두 번째 콜백 함수는 비동기 처리가 실패했을 때 호출되는 실패 처리 콜백 함수다.

```jsx
// fulfilled  
new Promise(resolve => resolve('fulfilled'))  
    .then(v => console.log(v), e => console.log(e)); // fulfilled  
  
// rejected  
new Promise((_, reject) => reject(new Error('rejected')))  
    .then(v => console.log(v), e => console.error(e)); // Error: rejected
```

then 메서드는 언제나 프로미스를 반환한다.

## 3.2 Promise.prototype.catch

catch 메서드는 한 개의 콜백 함수를 인수로 전달받는다.
catch 메서드의 콜백 함수는 프로미스가 rejected 상태인 경우만 호출된다.

```jsx
// rejected  
new Promise((_, reject) => reject(new Error('rejected')))  
  .catch(e => console.log(e)); // Error: rejected
```

catch 메서드는 then과 동일하게 동작한다.
-> then 메서드와 마찬가지로 언제나 프로미스를 반환한다.

```jsx
// rejected  
new Promise((_, reject) => reject(new Error('rejected')))  
  .then(undefined, e => console.log(e)); // Error: rejected
```

## 3.3 Promise.prototype.finally

finally 메서드는 한 개의 콜백 함수를 인수로 전달받는다.
finally 메서드의 콜백 함수는 프로미스의 성공 또는 실패와 상관없이 무조건 한 번 호출된다.
finally 메서드는 프로미스의 상태와 상관없이 공통적으로 수행해야 할 처리 내용이 있을 때 유용하다.
-> 마찬가지로 언제나 프로미스를 반환한다.

```jsx
// rejected  
new Promise(() => {})
  .finally(() => console.log('finally')); // finally
```

---
다음은 프로미스로 구현한 비동기 함수 get을 사용해 후속 처리를 한 것이다.

```jsx
const promiseGet = url => {  
  return new Promise((resolve, reject) => {  
    const xhr = new XMLHttpRequest();  
    xhr.open('GET', url);  
    xhr.send();  
  
    xhr.onload = () => {  
      if (xhr.status === 200) {  
        // 성공적으로 응답을 전달받으면 resolve 함수를 호출한다.  
        resolve(JSON.parse(xhr.response));  
      } else {  
        // 에러 처리를 위해 reject 함수를 호출한다.  
        reject(new Error(xhr.status));  
      }  
    };  
  });  
};  
  
// promiseGet 함수는 프로미스를 반환한다.  
promiseGet('https://jsonplaceholder.typicode.com/posts/1')  
  .then(res => console.log(res))  
  .catch(err => console.error(err))  
  .finally(() => console.log('Bye!'));
```

# 4. 프로미스의 에러 처리

비동기 함수 get은 프로미스를 반환한다. 비동기 처리 결과에 대한 후속 처리는 프로미스가 제공하는 후속 처리 메서드 then, catch, finally를 사용하여 수행한다.

비동기 처리에서 발생한 에러는 then 메서드의 두 번째 콜백 함수로 처리할 수 있다.

```jsx
const wrongUrl = 'https://jsonplaceholder.typicode.com/XXX/1';  
  
// 부적절한 URL이 지정되었기 때문에 에러가 발생한다.  
promiseGet(wrongUrl).then(  
  res => console.log(res),  
  err => console.error(err)  
); // Error: 404
```

비동기 처리에서 발생한 에러는 후속 처리 메서드 catch를 사용해 처리할 수도 있다.

```jsx
const wrongUrl = 'https://jsonplaceholder.typicode.com/XXX/1';  
  
// 부적절한 URL이 지정되었기 때문에 에러가 발생한다.  
promiseGet(wrongUrl)  
  .then(res => console.log(res))  
  .catch(err => console.error(err)); // Error: 404
```

catch 메서드를 호출하면 내부적으로 then(undefined, onRejected)을 호출한다.
catch 메서드를 모든 then 메서드를 호출한 이후에 호출하면 비동기 처리에서 발생한 에러뿐만 아니라 then 메서드 내부에서 발생한 에러까지 모두 캐치할 수 있다.

```jsx
const wrongUrl = 'https://jsonplaceholder.typicode.com/XXX/1';

promiseGet(wrongUrl)  
  .then(res => console.log(res))  
  .catch(err => console.error(err));
```

-> then 메서드에 두 번째 콜백 함수를 전달하는 것보다 catch 메서드를 사용하는 것이 가독성이 좋고 명확하다.

# 5. 프로미스 체이닝

```jsx
const url = 'https://jsonplaceholder.typicode.com';  
  
// id가 1인 post의 userId를 취득  
promiseGet(`${url}/posts/1`)  
  // 취득한 post의 userId로 user 정보를 취득  
  .then(({userId}) => promiseGet(`${url}/users/${userId}`))  
  .then(userInfo => console.log(userInfo))  
  .catch(err => console.error(err));
```

위 코드 후속 처리 메서드의 콜백 함수는 다음과 같이 인수를 전달받으면서 호출된다.

| **후속 처리 메서드** | **콜백 함수의 인수**                                                                   | **후속 처리 메서드의 반환값**                         |
| -------------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| then                 | promiseGet 함수가 반환한 프로미스가 resolve한 값(id가 1인 post)                        | 콜백 함수가 반환한 프로미스                           |
| then                 | 첫 번째 then 메서드가 반환한 프로미스가 resolve한 값(post의 userId로 취득한 user 정보) | 콜백 함수가 반환한 값(undefined)을 resolve한 프로미스 |
| catch                | promiseGet 함수 또는 앞선 후속 처리 메서드가 반환한 프로미스가 reject한 값             | 콜백 함수가 반환한 값(undefined)을 resolve한 프로미스                                                      |

프로미스는 프로미스 체이닝을 통해 비동기 처리 결과를 전달받아 후속 처리를 하므로 비동기 처리를 위한 콜백 패턴에서 발생하던 콜백 헬이 발생하지 않는다.

다만 프로미스도 콜백 패턴을 사용하므로 콜백 함수를 사용하지 않는 것은 아니다.

콜백 패턴은 가독성이 좋지않아 ES8에서 도입된 async/await를 통해 해결할 수 있다.

```jsx
const url = 'https://jsonplaceholder.typicode.com';  
  
(async () => {  
  // id가 1인 post의 userId를 취득  
  const {userId} = await promiseGet(`${url}/posts/1`);  
  // 취득한 post의 userId를 user 정보를 취득  
  const userInfo = await promiseGet(`${url}/users/${userId}`);  
  
  console.log(userInfo);  
})();
```

# 6.  프로미스의 정적 메서드

Promise는 5가지 정적 메서드를 제공한다.

## 6.1 Promise.resolve / Promise.reject

이미 존재하는 값을 래핑하여 프로미스를 사용하기 위해 사용한다.

- Promise.resolve 메서드는 인수로 전달받은 값을 resolve하는 프로미스를 생성한다.

```jsx
// 배열을 resolve하는 프로미스를 생성  
const resolvePromise = Promise.resolve([1, 2, 3]);  
// 위 코드가 아래와 같다.
const resolvePromise = new Promise(resolve => resolve([1, 2, 3]));  
resolvePromise.then(console.log); // [1, 2, 3]
```

- Promise.reject 메서드는 인수로 전달받은 값을 reject하는 프로미스를 생성한다.

```jsx
// 배열을 resolve하는 프로미스를 생성  
const rejectedPromise = Promise.reject(new Error('Error!'));  
// 위 코드가 아래와 같다.
const rejectedPromise = new Promise((_, reject) => reject(new Error('Error!'))); 
rejectedPromise.catch(console.log); // Error: Error!
```

## 6.2 Promise.all

Promise.all 메서드는 여러 개의 비동기 처리르 모두 병렬(parallel)처리할 때 사용한다.

```jsx
const requestData1 = () =>  
  new Promise(resolve => setTimeout(() => resolve(1), 3000));  
const requestData2 = () =>  
  new Promise(resolve => setTimeout(() => resolve(2), 2000));  
const requestData3 = () =>  
  new Promise(resolve => setTimeout(() => resolve(3), 1000));  
  
// 세 개의 비동기 처리를 순차적으로 처리  
const res = [];  
requestData1()  
  .then(data => {  
    res.push(data);  
    return requestData2();  
  })  
  .then(data => {  
    res.push(data);  
    return requestData3();  
  })  
  .then(data => {  
    res.push(data);  
    console.log(res); // [1, 2, 3] => 약 6초 소요  
  })  
  .catch(console.error);
```

위 예제는 세 개의 비동기 처리를 순차적으로 처리한다. -> 총 6초 이상이 소요된다.
그러나 위 예제의 경우 세 개의 비동기 처리는 서로 의존하지 않고 개별적으로 수행된다.
-> 순차적으로 처리할 필요가 없다. 병렬적으로 처리하려면 어떻게 해야 할까?
-> Promise.all 메서드를 활용하면 된다.

```jsx
const requestData1 = () =>  
  new Promise(resolve => setTimeout(() => resolve(1), 3000));  
const requestData2 = () =>  
  new Promise(resolve => setTimeout(() => resolve(2), 2000));  
const requestData3 = () =>  
  new Promise(resolve => setTimeout(() => resolve(3), 1000));  
  
// 세 개의 비동기 처리를 순차적으로 처리  
Promise.all([requestData1(), requestData2(), requestData3()])  
  .then(console.log) // [1, 2, 3] => 약 3초 소요  
  .catch(console.error);
```

Promise.all 메서드는 인수로 전달받은 배열의 모든 프로미스가 모두 fulfilled 상태가 되면 종료한다.
모든 프로미스가 fulfilled 상태가 되면 resolve된 처리 결과르 ㄹ모두 배열에 저장해 새로운 프로미스를 반환한다.

다른 프로미스가 가장 나중에 fulfilled 상태가 되어도 Promise.all 메서드는 첫 번째 프로미스가 resolve한 처리 결과부터 차례대로 배열에 저장해 그 배열을 resolve하는 새로운 프로미스를 반환한다.
-> 처리 순서가 보장된다.

Promise.all 메서드는 인수로 전달받은 배열의 프로미스가 하나라도 rejected 상태가 되면 나머지 프로미스가 fulfilled 상태가 되는 것을 기다리지 않고 즉시 종료한다.

```jsx
Promise.all([  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error 1')), 3000)),  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error 2')), 2000)),  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error 3')), 1000))  
])  
  .then(console.log)  
  .catch(console.error); // Error: Error 3
```

위 예제의 경우 세 번째 프로미스가 가장 먼저 rejected 상태가 되므로 세 번째 프로미스가 reject한 에러가 catch 메서드로 전달된다.

Promise.all 메서드는 인수로 전달받은 이터러블의 요소가 프로미스가 아닌 경우 Promise.resolve 메서드를 통해 프로미스로 래핑한다.

```jsx
Promise.all([  
  1, // -> Promise.resolve(1)  
  2, // -> Promise.resolve(2)  
  3  // -> Promise.resolve(3)  
])  
  .then(console.log) // [1, 2, 3]  
  .catch(console.error);
```

다음은 깃허브 아이디로 깃허브 사용자 이름을 취득하는 3개의 비동기 처리를 모두 병렬로 처리하는 코드다.

```jsx
// GET 요청을 위한 비동기 함수  
const promiseGet = url => {  
  return new Promise((resolve, reject) => {  
    const xhr = new XMLHttpRequest();  
    xhr.open('GET', url);  
    xhr.send();  
    xhr.onload = () => {  
      if (xhr.status === 200) {  
        // 성공적으로 응답을 전달받으면 resolve 함수를 호출한다.  
        resolve(JSON.parse(xhr.response));  
      } else {  
        // 에러 처리를 위해 reject 함수를 호출한다.  
        reject(new Error(xhr.status));  
      }  
    };  
  });  
};  
  
const githubIds = ['kses1010', 'sunny1234', 'cloud12'];  
  
Promise.all(githubIds.map(id => promiseGet(`https://api.github.com/users/${id}`)))  
  .then(users => users.map(user => user.name))  
  .then(console.log)  
  .catch(console.error)
```

## 6.3 Promise.race

Promise.race 메서드는 Promise.all 메서드와 동일하게 프로미스를 요소로 갖는 배열 등의 이터러블을 인수로 전달받는다.
Promise.race 메서드는 Promise.all 메서드처럼 모든 프로미스가 fulfilled 상태가 되는 것을 기다리는 것이 아니라 가장 먼저 fulfilled 상태가 된 프로미스의 처리 결과를 resolve하는 새로운 프로미스를 반환한다.

```jsx
Promise.race([  
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1  
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2  
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3  
])  
  .then(console.log) // 3  
  .catch(console.log);
```

프로미스가 rejected 상태가 되면 Promise.all 메서드와 동일하게 처리된다.
즉, Promise.race 메서드에 전달된 프로미스가 하나라도 rejected 상태가 되면 에러를 reject하는 새로운 프로미스를 즉시 반환한다.

```jsx
Promise.race([  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error 1')), 3000)),  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error 2')), 2000)),  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error 3')), 1000))  
])  
  .then(console.log)  
  .catch(console.log); // Error: Error 3
```

## 6.4 Promise.allSettled

Promise.allSettled 메서드는 프로미스를 요소로 갖는 배열 등의 이터러블을 인수로 전달받는다.
전달 받은 프로미스가 모두 settled 상태(fulfilled or rejected)가 되면 처리 결과를 배열로 반환한다.

```jsx
Promise.allSettled([  
  new Promise(resolve => setTimeout(() => resolve(1), 2000)),  
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error!')), 1000))  
]).then(console.log);
```

Promise.allSettled 메서드가 반환한 배열에는 fulfilled or rejected 상태와는 상관없이 인수로 전달받은 모든 프로미스들의 처리 결과가 모두 담겨 있다.
- fulfilled: 비동기 처리 상태 status 프로퍼티와 처리 결과를 나타내는 value 프로퍼티를 갖는다.
- rejected: 비동기 처리 상태 status 프로퍼티와 에러를 나타내는 reason 프로퍼티를 갖는다.

```json
[  
  // 프로미스가 fulfilled 상태인 경우  
  {status: "fulfilled", value: 1},  
  // 프로미스가 rejected 상태인 경우  
  {status: "rejected", reason: Error: Error! at <anonymous>:3:60}  
]
```

# 7. 마이크로태스크 큐

```jsx
setTimeout(() => console.log(1), 0);  
  
Promise.resolve()  
  .then(() => console.log(2))  
  .then(() => console.log(3));
```

프로미스의 후속 처리 메서드도 비동기로 동작하므로 `1 -> 2 -> 3`의 순으로 출력될 것처럼 보이지만 
`2 -> 3 -> 1` 의 순으로 출력된다.
그 이유는 프로미스의 후속 처리 메서드의 콜백 함수는 태스크 큐가 아니라 마이크로태스크 큐에 저장되기 때문이다.
마이크로태스크 큐에는 프로미스의 후속 처리 메서드의 콜백 함수가 일시 저장된다.
**마이크로태스크 큐는 태스크 큐보다 우선순위가 높다.**

즉, 이벤트 루프는 콜 스택이 비면 먼저 마이크로태스크 큐에서 대기하고 있는 함수를 가져와 실행한다.
이후 마이크로태스크 큐가 비면 태스크 큐에서 대기하고 있는 함수를 가져와 실행한다.

# 8. fetch

fetch 함수는 XMLHttpRequest 객체와 마찬가지로 HTTP 요청 전송 기능을 제공하는 클라이언트 사이드 Web API다.
fetch 함수는 XMLHttpRequest 객체보다 사용법이 간단하고 프로미스를 지원하기 때문에 비동기 처리를 위한 콜백 패턴의 단점에서 자유롭다.

**fetch 함수는 HTTP 응답을 나타내는 Response 객체를 래핑한 Promise 객체를 반환한다.**

```jsx
fetch('https://jsonplaceholder.typicode.com/todos/1')
  .then(response => console.log(response));
```

fetch 함수는 HTTP 응답을 나타내는 Response 객체를 래핑한 프로미스를 반환하므로 후속 처리 메서드 then을 통해 프로미스가 resolve한 Response 객체를 전달받을 수 있다.

```jsx
fetch('https://jsonplaceholder.typicode.com/todos/1')  
  // response는 HTTP 응답을 나타내는 Response 객체다.  
  // json 메서드를 사용하여 Response 객체에서 HTTP 응답 몸체를 취득하여 역직렬화한다.  
  .then(response => response.json())  
  // json은 역직렬화 HTTP 응답 몸체다.  
  .then(json => console.log(json));
```

fetch 함수를 통해 HTTP 요청을 전송하면 다음과 같다.
fetch 함수에 첫 번째 인수로 HTTP 요청을 전송할 URL과 두 번째 인수로 HTTP 요청 메서드, HTTP 요청 헤더, 페이로드 등을 설정한 객체를 전달한다.

```jsx
const request = {  
  get(url) {  
    return fetch(url);  
  },  
  post(url, payload) {  
    return fetch(url, {  
      method: 'POST',  
      headers: {'content-Type' : 'application/json'},  
      body: JSON.stringify(payload)  
    });  
  },  
  patch(url, payload) {  
    return fetch(url, {  
      method: 'PATCH',  
      headers: {'content-Type' : 'application/json'},  
      body: JSON.stringify(payload)  
    });  
  },  
  delete(url) {  
    return fetch(url, {method: 'DELETE'});  
  }  
};
```

### 1. GET 요청

```jsx
request.get('https://jsonplaceholder.typicode.com/todos/1')  
  .then(response => response.json())  
  .then(todos => console.log(todos))  
  .catch(err => console.error(err));
```

### 2. POST 요청

```jsx
request.post('https://jsonplaceholder.typicode.com/todos', {  
  userId: 1,  
  title: 'Javascript',  
  completed: false  
}).then(response => response.json())  
  .then(todos => console.log(todos))  
  .catch(err => console.error(err));
```

### 3. PATCH 요청

```jsx
request.patch('https://jsonplaceholder.typicode.com/todos', {  
    completed: true  
}).then(response => response.json())  
  .then(todos => console.log(todos))  
  .catch(err => console.error(err));
```

### 4. DELETE 요청

```jsx
request.delete('https://jsonplaceholder.typicode.com/todos/1')  
  .then(response => response.json())  
  .then(todos => console.log(todos))  
  .catch(err => console.error(err));
```
