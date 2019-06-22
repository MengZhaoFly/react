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
