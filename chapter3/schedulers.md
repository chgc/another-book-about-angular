# Schedulers

## Scheduler 相關 operators

### observeOn

設定 Observable 的 scheduler 模式
**使用介面**

```typescript
observeOn(scheduler: *, delay: *): Observable<R> | WebSocketSubject<T> | Observable<T>
```

**使用範例**

```typescript
Observable.of(1, 2, 3)
          .observeOn(async)
          .subscribe(value => console.log(value));.
```

### subscribeOn

設定 subscribe 的 scheduler 模式
![](images/subscribeOn.png)
**使用介面**

```typescript
subscribeOn(scheduler: Scheduler): Observable<T>
```

**使用模式**

```typescript
Observable.of(1, 2, 3)
          .subscribeOn(async)
          .subscribe(value => console.log(value));
```

# 