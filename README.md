# $try

## Inline use

```ts
function computation(a: number, b: number) {
  if (Math.random() > 0.5) throw new CustomError('failed computation');
  return a + b
}

const [result, error] = $try(computation)(2, 3);

// With type error
const [result, error] = $try(computation)<CustomError>(2, 3);

if (error) {
  // handle the error
}
// handle the success
```

## 

```ts
const computation = $try((a: number, b: number) => {
  if (Math.random() > 0.5) throw new CustomError('failed computation');
  return a + b
})<CustomError>;

const [result, error] = computation(2, 3);

if (error) {
  // handle the error
}
// handle the success
```

## Source code

```ts
type AnyFunction = (...args: any) => any | Promise<any>;

type ResultSuccess<T> = [T, null];
type ResultFail<Err> = [null, Err];
type Optional<Data, Err> = ResultSuccess<Data> | ResultFail<Err>;

type Result<T, Err> = T extends Promise<any> ? Promise<Optional<Awaited<T>, Err>> : Optional<T, Err>;

function $try<T extends AnyFunction>(fn: T) {
  return function <Err = Error>(...args: Parameters<T>): Result<ReturnType<T>, Err> {
    try {
      const result = fn(...args);
      if (isPromise(result)) {
        const res = result.then((val) => [val, null]).catch((err) => [null, err]);
        return res as Result<ReturnType<T>, Err>;
      }
      return [result, null] as Result<ReturnType<T>, Err>;
    } catch (e) {
      return [null, e] as Result<ReturnType<T>, Err>;
    }
  };
}
```
