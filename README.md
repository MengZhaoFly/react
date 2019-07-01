## React.childRen.map(children, callback)

1. let result = []
2. 第一步调用 mapIntoWithKeyPrefixInternal 
   会先从对象池里面取出一个对象，把 result 和 callback 都交给它，
3. 会拍平数组
    1. 全靠 `traverseAllChildrenImpl `发力， 递归 了一遍 children，
      如果符合

      ```js

      switch (type) {
         case 'string':
         case 'number':
           invokeCallback = true;
           break;
         case 'object':
           switch (children.$$typeof) {
             case REACT_ELEMENT_TYPE:
             case REACT_PORTAL_TYPE:
               invokeCallback = true;
           }
       }
       ```
       则会 调用 callback

4. callback 执行 发现 返回的结果还是 数组 再一次递归 mapIntoWithKeyPrefixInternal，
   如果不是则会放在 result 最后返回。

## hydrate && render
是否调和原先存在 container 里面的节点，假设 container 里面存在节点，react 就会调和
```js
const shouldHydrate =
  forceHydrate || shouldHydrateDueToLegacyHeuristic(container);

function shouldHydrateDueToLegacyHeuristic(container) {
  const rootElement = getReactRootElementInContainer(container);
  return !!(
    rootElement &&
    rootElement.nodeType === ELEMENT_NODE &&
    rootElement.hasAttribute(ROOT_ATTRIBUTE_NAME)
  );
}
// export const ROOT_ATTRIBUTE_NAME = 'data-reactroot';
function getReactRootElementInContainer(container: any) {
  if (!container) {
    return null;
  }

  if (container.nodeType === DOCUMENT_NODE) {
    return container.documentElement;
  } else {
    return container.firstChild;
  }
}
```

## render
1. ```js
    legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    ```
    创建一个 FiberRoot
2. updateContainer 创建 expirationTime
3. 创建更新，调度更新
 ```js
  [enqueueUpdate && scheduleWork](https://github.com/MengZhaoFly/react/blob/master/packages/react-reconciler/src/ReactFiberReconciler.js#L170)
  enqueueUpdate(current, update);
  scheduleWork(current, expirationTime);
  ```

## update && updateQueue
1. react/packages/react-reconciler/src/ReactFiberReconciler.js
   [createUpdate()](https://github.com/MengZhaoFly/react/blob/master/packages/react-reconciler/src/ReactFiberReconciler.js#L150)
   ```js
   createUpdate()
   export type Update<State> = {
     // 更新过期时间
      expirationTime: ExpirationTime,
      suspenseConfig: null | SuspenseConfig,
      // 0 updateState 1 replaceState 2 forceUpdate 3 captureUpdate
      tag: 0 | 1 | 2 | 3,
      payload: any,
      callback: (() => mixed) | null,
      // 下一个更新 update 存在 updateQueue 里面 类似单向列表
      next: Update<State> | null,
      nextEffect: Update<State> | null,
    };
   ```
2. updateQueue 的结构
```js
  export type UpdateQueue<State> = {
  baseState: State,

  firstUpdate: Update<State> | null,
  lastUpdate: Update<State> | null,

  firstCapturedUpdate: Update<State> | null,
  lastCapturedUpdate: Update<State> | null,

  firstEffect: Update<State> | null,
  lastEffect: Update<State> | null,

  firstCapturedEffect: Update<State> | null,
  lastCapturedEffect: Update<State> | null,
  };
```
3. enqueueUpdate
   创建 或者 更新 updateQueue

## 计算 ExpirationTime
异步任务优先级比较低，可能会被打断，因为会被打断，为了防止一直被打断，所以设置 expirationTime,在这个时间之前都可以被打断，过了这个时间就会强制执行。
更新的一个过期时间
```js
// 整理了一个文件
// react/packages/react-reconciler/src/ReactFiberExpirationTime.html
/* bucketSizeMs 100
expirationInMs 500 
UNIT_SIZE 10
*/
运行：computeInteractiveExpiration(102) ～ computeInteractiveExpiration(111) 得到的结果都是一样的。
function ceiling(num, precision) {
  return (((num / precision) | 0) + 1) * precision;
}
function computeExpirationBucket(
  currentTime,
  expirationInMs,
  bucketSizeMs,
) {
  return (
    MAGIC_NUMBER_OFFSET -
    ceiling(
      MAGIC_NUMBER_OFFSET - currentTime + expirationInMs / UNIT_SIZE,
      bucketSizeMs / UNIT_SIZE,
    )
  );
}
原因：precision 固定为 10， 导致 在进行 num / precision | 0 运算时，只要两个数之间的差距不超过10，得到的结果都一样，这样得到一个相同的 expirationtime 的好处是，在进行 大量的 setState 时候，某一很小的时间间距内段内的expirationtime一样。
```
## 不同的 expirationTime
sync模式
异步模式
指定 context
[computeExpirationForFiber](https://github.com/MengZhaoFly/react/blob/master/packages/react-reconciler/src/ReactFiberWorkLoop.js#L278)
根据优先级和fiber.mode 判断到底该用哪个时间

## setState && forceUpdate
给节点的 Fiber 创建更新
更新的类型不同
流程：
1. 添加tag
2. 创建更新
3. 进入 scheduleWork
```js
enqueueForceUpdate(inst, callback) {
    const fiber = getInstance(inst);
    const currentTime = requestCurrentTime();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig,
    );
    const update = createUpdate(expirationTime, suspenseConfig);
    update.tag = ForceUpdate;

    if (callback !== undefined && callback !== null) {
      update.callback = callback;
    }

    if (revertPassiveEffectsChange) {
      flushPassiveEffects();
    }
    enqueueUpdate(fiber, update);
    scheduleWork(fiber, expirationTime);
  },
```


