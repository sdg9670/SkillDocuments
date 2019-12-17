# JavaScript 객체 복사

## 들어가기 앞서...

아래와 같은 경우에는 객체가 복사되지 않았습니다. 그 이유는 객체 내부의 변수의 값을 변경하면 복사한 데이터의 값도 바뀌기 때문입니다. 이는 단지 `참조`하는 주소를 대입한 것 입니다. 아래와 같이 예측 불가능한 상태 변화로 인한 오류가 빈번합니다.

```javascript
var object = {a: 10};
var copy_object = object;
object.a = 7;
console.log("object", object.a);
console.log("copy", copy_object.a);
console.log(object == copy_object);

/* 출력
object 7
copy 7
true
*/
```


## 얕은 복사

얕은 복사란 객체 내부의 값은 복사되었는데 내부의 객체 값은 복사되지 않는 경우입니다. 아래의 경우를 살펴보면 내부 객체의 값이 일괄적으로 변한 것을 확인할 수 있습니다.

- **Object.assign**

```javascript
var object = {a: 10, b:{c: 15}};
var copy_object = Object.assign({}, object);
object.a = 7;
object.b.c = 3;
console.log(object == copy_object);
console.log("object.a:", object.a, "copy.a:", copy_object.a);
console.log("object.b.c:", object.b.c, "copy.b.c:", copy_object.b.c);
/* 출력
false
object.a: 7 copy.a: 10
object.b.c: 3 copy.b.c: 3
*/
```

- **Spread**

```javascript
var object = {a: 10, b:{c: 15}};
var copy_object = {...object}
object.a = 7;
object.b.c = 3;
console.log(object == copy_object);
console.log("object.a:", object.a, "copy.a:", copy_object.a);
console.log("object.b.c:", object.b.c, "copy.b.c:", copy_object.b.c);
/* 출력
false
object.a: 7 copy.a: 10
object.b.c: 3 copy.b.c: 3
*/
```

## 깊은 복사

깊은 복사는 얕은 복사와는 달리 객체 내부의 객체의 상태도 복사하는 것입니다. 아래의 경우를 보면 모든 데이터가 복사되어 값을 변경하여도 문제없이 예측한대로 작동합니다. 깊은 복사의 경우 `외부라이브러리`를 사용하는 방법, `객체를 탐색`해서 복사하는 방법, `Json 함수`를 이용하는 방법이 있습니다. 객체 탐색과 JSON 함수를 사용해서 작동하는 예제와 복사 시간 비교를 해봤습니다. 결과적으로는 JSON 함수가 약 `15배`정도 더 느렸습니다.

- **객체 탐색**

```javascript
function deepCopyObject(inObject) {
  var outObject, value, key
  if(typeof inObject !== "object" || inObject === null) {
    return inObject
  }
  outObject = Array.isArray(inObject) ? [] : {}
  for (key in inObject) {
    value = inObject[key]
    outObject[key] = (typeof value === "object" && value !== null) ? deepCopyObject(value) : value
  }
  return outObject
}

var obj = {a: {b: {c: {d: {e: "kr"}}}}}
var sumTime = 0;
var trys = 10;

for(let t = 0; t < trys; t++)
{
  var startTime = new Date().getTime();

  for(let i = 0; i < 10000000; i++)
  {
    var newObj = deepCopyObject(obj);
  }

  var endTime = new Date().getTime();

  console.log(t+1, endTime - startTime);
  sumTime += endTime - startTime;
}
console.log("평균", sumTime/trys);

/* 결과
1 1629
2 1618
3 1592
4 1587
5 1582
6 1576
7 1592
8 1570
9 1570
10 1637
평균 1595.3
*/
```

- **JSON 함수**

```javascript
var object = {a: {b: {c: {d: {e: "kr"}}}}}
var sumTime = 0;
var trys = 10;

for(let t = 0; t < trys; t++)
{
  var startTime = new Date().getTime();

  for(let i = 0; i < 10000000; i++)
  {
    var newObj = JSON.parse(JSON.stringify(object));
  }

  var endTime = new Date().getTime();

  console.log(t+1, endTime - startTime);
  sumTime += endTime - startTime;
}
console.log("평균", sumTime/trys);

/*
결과
1 21311
2 21121
3 21631
4 21915
5 21586
6 21724
7 21831
8 21623
9 21647
10 21509
평균 21589.8
*/
```

## 결과적으로...

위의 방법으로 코딩을 하면 무의식적으로 코딩한 것 보다 예측 불가능한 상태변화를 피할 수 있어 오류 발생률이 낮아질 것 입니다. 이러한 문제를 해결하기위해 페이스북에서는 [`ImmutableJS`](https://github.com/immutable-js/immutable-js)라는 라이브러리를 만들었으니 참고해보시는 것도 좋을 것 같습니다.