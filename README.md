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
3. ```js
  [enqueueUpdate && scheduleWork](https://github.com/MengZhaoFly/react/blob/master/packages/react-reconciler/src/ReactFiberReconciler.js#L170)
  enqueueUpdate(current, update);
  scheduleWork(current, expirationTime);
  ```

  创建更新，调度更新

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

