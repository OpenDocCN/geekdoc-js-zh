# Observable 封装

# Observable 封装

我们刚刚在 Observable 解剖中学到，关键操作 `next()`、`error()` 和 `complete` 是让我们的 Observable 运转的要素，如果我们自己定义了它。我们还学到了，这些方法会在我们的订阅中触发相应的回调。

将某物封装到 Observable 中意味着我们将一个不是 Observable 的东西转变为一个 Observable，这样它就可以与其他 Observables 协同工作。这也意味着它现在可以使用操作符。

## 封装一个 Ajax 调用

```
let stream = Rx.Observable.create((observer) => {
   let request = new XMLHttpRequest();

   request.open( ‘GET’, ‘url’ );
   request.onload =() =>{
      if(request.status === 200) {
         observer.next( request.response );
         observer.complete();
     } else {
          observer.error('error happened');
     }
   }

   request.onerror = () => {  
       observer.error('error happened')                                                                                } 
   request.send();
})

stream.subscribe(
   (data) => console.log( data )  
) 
```

这里我们需要做三件事情：`发射数据`、`处理错误` 和 `关闭流`

### 发射数据

```
if(request.status === 200) {
  observer.next( request.response )  // emit data

} 
```

### 处理可能的错误

```
else {
       observer.error('error happened');
  } 
```

并且

```
request.onerror = () => {
   observer.error('error happened') 
} 
```

### 关闭流

```
if(request.status === 200) {
  observer.next( request.response )
  observer.complete()  // close stream, as we don't expect more data
} 
```

## 封装语音音频 API

```
console.clear();

const { Observable } = Rx;

const speechRecognition$ = new Observable(observer => {
   const speech = new webkitSpeechRecognition();

   speech.onresult = (event) => {
     observer.next(event);
     observer.complete();
   };

   speech.start();

   return () => {
     speech.stop();
   }
});

const say = (text) => new Observable(observer => {
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.onend = (e) => {
    observer.next(e);
    observer.complete();
  };
  speechSynthesis.speak(utterance);
});

const button = document.querySelector("button");

const heyClick$ = Observable.fromEvent(button, 'click');

heyClick$
  .switchMap(e => speechRecognition$)
  .map(e => e.results[0][0].transcript)
  .map(text => {
    switch (text) {
      case 'I want':
        return 'candy';
      case 'hi':
      case 'ice ice':
        return 'baby';
      case 'hello':
        return 'Is it me you are looking for';
      case 'make me a sandwich':
      case 'get me a sandwich':
        return 'do it yo damn self';
      case 'why are you being so sexist':
        return 'you made me that way';
      default:
        return `I don't understand: "${text}"`;
    }
  })
  .concatMap(say)
  .subscribe(e => console.log(e)); 
```

### 语音识别流

这会激活浏览器中的麦克风并录制我们的声音

```
const speechRecognition$ = new Observable(observer => {
   const speech = new webkitSpeechRecognition();

   speech.onresult = (event) => {
     observer.next(event);
     observer.complete();
   };

   speech.start();

   return () => {
     speech.stop();
   }
}); 
```

这基本上设置了语音识别 API。我们等待一个响应，之后我们完成流，就像第一个例子中的 AJAX 一样。

还要注意，为清理而定义了一个函数

```
return () => {
   speech.stop();
 } 
```

这样我们就可以调用 `speechRecognition.unsubscribe()` 来清理资源

### 语音合成发声，say

这负责发出你想要它发出的声音（say）。

```
const say = (text) => new Observable(observer => {
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.onend = (e) => {
    observer.next(e);
    observer.complete();
  };
  speechSynthesis.speak(utterance);
}); 
```

### 主流 hey$

```
heyClick$
  .switchMap(e => speechRecognition$)
  .map(e => e.results[0][0].transcript)
  .map(text => {
    switch (text) {
      case 'I want':
        return 'candy';
      case 'hi':
      case 'ice ice':
        return 'baby';
      case 'hello':
        return 'Is it me you are looking for';
      case 'make me a sandwich':
      case 'get me a sandwich':
        return 'do it yo damn self';
      case 'why are you being so sexist':
        return 'you made me that way';
      default:
        return `I don't understand: "${text}"`;
    }
  })
  .concatMap(say)
  .subscribe(e => console.log(e)); 
```

逻辑应该如下阅读 `heyClick$` 在按钮点击时被激活。`speechRecognition` 在监听我们说的话，并将结果发送到 `heyClick$`，其中切换逻辑确定了一个适当的响应，由 `say` Observable 发出。

非常感谢 @ladyleet 和 @benlesh

## 总结

一个更简单的 Ajax 封装和一个稍微高级一点的 Speech API 已经被封装到一个 Observable 中。尽管机制仍然相同：1) 在数据被发射的地方，添加一个调用 `next()` 的命令 2) 如果没有更多数据需要发射，调用 `complete` 3) 如果有需要，定义一个可以在 `unsubscribe()` 中调用的函数 4) 通过在适当的位置调用 `.error()` 来处理错误。（仅在第一个示例中完成）
