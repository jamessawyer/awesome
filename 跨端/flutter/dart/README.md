事件循环:
dart事件循环和js事件循环几乎一样

:sailboat: dart版本：

```dart
import 'dart:async';

void main() {
  print('main-A');
  
  // 宏任务
  Timer.run(() {
    print('B');
    
    // 微任务
    scheduleMicrotask(() {
      print('B-m-1')
    });
    
    scheduleMicrotask(() {
      print('B-m-2')
    });
  });
  
  // 微任务
  scheduleMicrotask(() {
    print('C');
    
    // 宏任务
    Timer.run(() {
      print('C-t-1')
    });
  });
  
  print('main-D');
}
```

打印结果：

```bash
main-A
main-D
C
B
B-m-1
B-m-2
C-t-1
```

:sake: js版本：

```js
function main() {
  console.log('main-A')
  
  // 宏任务
  setTimeout(() => {
    console.log('B')
    
    // 微任务
    Promise.resolve().then(() => console.log('B-m-1'))
    Promise.resolve().then(() => console.log('B-m-2'))
  }, 0)
  
  // 微任务
  Promise.resolve().then(() => {
    console.log('C')
    
    // 宏任务
    setTimeout(() => console.log('C-t-1'), 0)
  })
  
  console.log('main-D')
}

main()
```

打印结果：

```bash
main-A
main-D
C
B
B-m-1
B-m-2
C-t-1
```

由上可以看出，事件循环机制整体执行顺序是：

1. 先将同步任务加入到调用栈（call stack），并执行出栈，打印 `main-A`
2. 碰到宏任务，将宏任务放入 **宏任务队列中**， 即 setTimeout
3. 碰到微任务，将微任务放入 **微任务队列**， 即 Promise
4. 又碰到同步任务，加入调用栈，执行出栈，打印 `main-D`
5. 由于 **微任务队列优先级高于宏任务队列**，事件循环先将微任务队列中的任务加入到调用栈中，碰到同步任务，打印 `C`，随之是一个setTimeout宏任务，将其加入到宏任务队列中
6. 调用栈为空，再次事件循环，此时微任务队列为空，执行宏任务队列，宏任务队列第一个任务进入调用栈，打印 `B` 随之是2个Promise 微任务，将其依次加入到微任务队列中，即 B-m-1 & B-m-2 进入微任务队列
7. 调用栈为空，再次事件循环，**每次都是优先检查微任务队列是否存在任务**，因为步骤6中加入了2个微任务，依次执行这2个微任务，依次打印 `B-m-1` & `B-m-2`
8. 调用栈为空，再次事件循环，此时微任务队列也为空，执行宏任务队列，将宏任务队列中的任务加入到调用栈中，打印 `C-t-1`

整个步骤的可视化可以查看：

- [jsv9000.app event loop](https://www.jsv9000.app/?code=ZnVuY3Rpb24gQSgpIHsgY29uc29sZS5sb2coJ21haW4tQScpIH0KZnVuY3Rpb24gQigpIHsgY29uc29sZS5sb2coJ0InKSB9CmZ1bmN0aW9uIEJNMSgpIHsgY29uc29sZS5sb2coJ0ItbS0xJykgfQpmdW5jdGlvbiBCTTIoKSB7IGNvbnNvbGUubG9nKCdCLW0tMicpIH0KZnVuY3Rpb24gQygpIHsgY29uc29sZS5sb2coJ0MnKSB9CmZ1bmN0aW9uIENUMSgpIHsgY29uc29sZS5sb2coJ0MtdC0xJykgfQpmdW5jdGlvbiBEKCkgeyBjb25zb2xlLmxvZygnbWFpbi1EJykgfQoKQSgpCnNldFRpbWVvdXQoKCkgPT4gewogIEIoKQogIFByb21pc2UucmVzb2x2ZSgpLnRoZW4oQk0xKQogIFByb21pc2UucmVzb2x2ZSgpLnRoZW4oQk0yKQp9LCAwKQoKUHJvbWlzZS5yZXNvbHZlKCkudGhlbigoKSA9PiB7CiAgQygpCiAgc2V0VGltZW91dChDVDEsIDApCn0pCgpEKCkK)
- [dart 版本 Dart 单线程（事件轮询），忙了两天，80帧动画，全景展示运行机制 - whoicliu@bilibili](https://www.bilibili.com/video/BV1WR4y1H7Lx?from=search&seid=1019915141032428511&spm_id_from=333.337.0.0)
