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
  enqueueUpdate(current, update);
  scheduleWork(current, expirationTime);
  ```

  创建更新，调度更新
