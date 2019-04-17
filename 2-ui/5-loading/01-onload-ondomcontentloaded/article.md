<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
# 页面生命周期：DOMContentLoaded、load、beforeunload 和 unload
=======
# Page: DOMContentLoaded, load, beforeunload, unload
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

HTML 页面的生命周期有三个重要事件：

- `DOMContentLoaded` —— 浏览器加载 HTML，并构建 DOM 树，但像 `<img>` 和样式这样的资源可能还没有加载。
- `load` —— 浏览器加载所有资源（图像，样式等）。
- `beforeunload/unload` —— 当用户离开页面时。

每个事件都是有用的：

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
- `DOMContentLoaded` 事件 —— DOM 已经准备好，因此处理器可以查找 DOM 节点，并初始化接口。
- `load` 事件 —— 额外资源被加载后，我们可以获取图像大小（如果在 HTML/CSS 中没有指定）等。
- `beforeunload/unload` 事件 —— 用户即将离开：我们可以检查用户是否保存了修改，并在询问他是否真的要离开。
=======
- `DOMContentLoaded` event -- DOM is ready, so the handler can lookup DOM nodes, initialize the interface.
- `load` event -- additional resources are loaded, we can get image sizes (if not specified in HTML/CSS) etc.
- `beforeunload` event -- the user is leaving: we can check if the user saved the changes and ask them whether they really want to leave.
- `unload` -- the user almost left, but we still can initiate some operations, such as sending out statistics.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

我们探讨一下这些事件的细节。

## DOMContentLoaded

`DOMContentLoaded` 事件发生在 `document` 对象上。

我们必须使用 `addEventListener` 来监听它：

```js
document.addEventListener("DOMContentLoaded", ready);
// not "document.onDOMContentLoaded = ..."
```

例如：

```html run height=200 refresh
<script>
  function ready() {
    alert('DOM is ready');

    // 图像尚未加载（除非已经有了缓存）因此大小是 0x0
    alert(`Image size: ${img.offsetWidth}x${img.offsetHeight}`);
  }

*!*
  document.addEventListener("DOMContentLoaded", ready);
*/!*
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
在示例中，`DOMContentLoaded` 处理器在文档加载时运行，而不是等到页面被加载时运行。因此 `alert` 显示大小为零。

初识 `DOMContentLoaded` 事件时，觉得它比较简单。DOM 树已经准备好了 —— 这是事件。但却没有什么特别之处。
=======
In the example the `DOMContentLoaded` handler runs when the document is loaded, so it can see all the elements, including `<img>` below.

But it doesn't wait for the image to load. So `alert` shows zero sizes.

At the first sight `DOMContentLoaded` event is very simple. The DOM tree is ready -- here's the event. There are few peculiarities though.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

### DOMContentLoaded 和脚本

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
当浏览器开始加载 HTML 并在文本中遇到 `<script>...</script>` 时，就会停止构建 DOM。它必须立即执行脚本。因此 `DOMContentLoaded` 只有在所有此类脚本被执行后才会发生。

额外的脚本（带有 `src`）也会促使 DOM 构建在加载和执行过程时暂停。因此 `DOMContentLoaded` 也会等待外部脚本。

唯一的例外是具有 `async` 和 `defer` 属性的外部脚本。它们告诉浏览器可以继续解析文档而不必等待脚本解析和执行。因此用户可以在脚本完成加载之前就看到页面，这对性能来说是有好处的。

```smart header="A word about `async` and `defer`"
属性 `async`和 `defer` 仅适用于外部脚本。如果没有 `src`，它们就会被忽略。

这两种方法告诉浏览器，它可以继续解析页面，并“在后台”继续加载脚本，然后在外部脚本加载完成后执行它。因此脚本不会阻塞 DOM 的构建和页面的渲染。

他们之间有两个不同之处。

|         | `async` | `defer` |
|---------|---------|---------|
| Order | 具有 `async` 的脚本以**第一顺序被加载**。它们的文档顺序并不重要 —— 先加载先运行。 | 具有 `defer` 的脚本总是按照**文档顺序**来执行（就像它们在文档中那样）。 |
| `DOMContentLoaded` | 具有 `async` 的脚本可以在文档尚未完全下载时加载和执行。如果脚本较小或被缓存，而且文档足够长，就会发生这种情况。 | 在 `DOMContentLoaded` 之前，具有 `defer` 的脚本会在文档被加载并解析后执行（如果需要，它们会等待）。 |

因此 `async` 用于完全独立的脚本。
=======
When the browser processes an HTML-document and comes across a `<script>` tag, it needs to execute before continuing building the DOM. That's a precaution, as scripts may want to modify DOM, and even `document.write` into it, so `DOMContentLoaded` has to wait.

So DOMContentLoaded definitely happens after such scripts:

```html run
<script>
  document.addEventListener("DOMContentLoaded", () => {
    console.log("DOM ready!");
  });
</script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js"></script>

<script>
  alert("Library loaded, inline script executed");
</script>
```

In the example above, we first see "Library loaded...", and then "DOM ready!" (all scripts are executed).

```warn header="Scripts with `async`, `defer` or `type=\"module\"` don't block DOMContentLoaded"

Script attributes `async` and `defer`, that we'll cover [a bit later](info:script-async-defer), don't block DOMContentLoaded. [Javascript modules](info:modules) behave like `defer`,  they don't block it too.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

So here we're talking about "regular" scripts, like `<script>...</script>`, or `<script src="..."></script>`.
```

### DOMContentLoaded 和样式

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
外部样式不会影响 DOM，因此 `DOMContentLoaded` 无需等待它们。

但有一个陷阱：如果在样式之后有一个脚本，那么该脚本必须等待样式被执行：
=======
External style sheets don't affect DOM, so `DOMContentLoaded` does not wait for them.

But there's a pitfall. If we have a script after the style, then that script must wait until the stylesheet loads:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

```html
<link type="text/css" rel="stylesheet" href="style.css">
<script>
  // the script doesn't not execute until the stylesheet is loaded
  alert(getComputedStyle(document.body).marginTop);
</script>
```

原因是脚本可能希望获取如上述示例所描述的元素坐标和其他与样式相关的属性。当然，它必须等待样式被加载。

当 `DOMContentLoaded` 等待脚本时，它也在等待它们之前的样式。

### 浏览器内置填写

Firefox、Chrome 和 Opera 都会在 `DOMContentLoaded` 中自动填写表单。

比如，如果页面有一个带有登录和密码的表单，并且浏览器记住了这些值，那么在 `DOMContentLoaded` 上，它就可以尝试自动填写它们（如果用户允许）。

因此如果 `DOMContentLoaded` 被长加载脚本延迟，那么自动填写也在等待。你可能在某些站点上（如果你使用浏览器自动填写）—— 登录/密码字段将不会立即自动填写，在页面被完全加载前会出现延迟。这实际上是延迟到 `DOMContentLoaded` 事件。

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
为外部脚本使用 `async` 和 `defer` 的一个好处是 —— 它们不会阻塞 `DOMContentLoaded`，而且也不会延迟浏览器的自动填写。
=======
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

## window.onload [#window-onload]

当包括样式、图像和其他资源的页面被全部加载时，`load` 事件就会在 `window` 对象上被触发。

以下示例正确地显示了图像大小，因为 `window.onload` 等待了所有的图像：

```html run height=200 refresh
<script>
  window.onload = function() {
    alert('Page loaded');

    // image is loaded at this time
    alert(`Image size: ${img.offsetWidth}x${img.offsetHeight}`);
  };
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

## window.onunload

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
当访问者离开页面时，`unload` 事件会在 `window` 上被触发。我们可以在那里做一些不涉及延迟的事件，比如关闭相关的弹出窗口。但我们不能取消跳转到另一个页面的事件。

因此我们需要使用另一个事件 —— `onbeforeunload`。
=======
When a visitor leaves the page, the `unload` event triggers on `window`. We can do something there that doesn't involve a delay, like closing related popup windows.

The notable exception is sending analytics.

Let's say we gather data about how the page is used: mouse clicks, scrolls, viewed page areas, and so on.

Naturally, `unload` event is when the user leaves us, and we'd like to save the data on our server.

There exists a special `navigator.sendBeacon(url, data)` method for such needs, described in the specification <https://w3c.github.io/beacon/>.

It sends the data in background. The transition to another page is not delayed: the browser leaves the page, but still performs `sendBeacon`.

Here's how to use it:
```js
let analyticsData = { /* object with gathered data */ };

window.addEventListener("unload", function() {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
};
```

- The request is sent as POST.
- We can send not only a string, but also forms and other formats, as described in the chapter <info:fetch-basics>, but usually it's a stringified object.
- The data is limited by 64kb.

When the `sendBeacon` request is finished, the browser probably has already left the document, so there's no way to get server response (which is usually empty for analytics).

There's also a `keepalive` flag for doing such "after-page-left" requests in  [fetch](info:fetch-basics) method for generic network requests. You can find more information in the chapter <info:fetch-api>.


If we want to cancel the transition to another page, we can't do it here. But we can use  another event -- `onbeforeunload`.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

## window.onbeforeunload [#window.onbeforeunload]

如果访问中启动了离开页面的导航或试图关闭窗口，`beforeunload` 处理器将要求提供更多的确认。

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
它可能会返回一个带有问题的字符串。从历史上看，浏览器通常会显示它，但到目前为止，只有一些浏览器这样做。这是因为某些站长滥用了这个事件处理器，显示了误导和恶意的信息。

你可以通过运行这段代码，然后重新加载页面来进行尝试。
=======
If we cancel the event, the browser may ask the visitor if they are sure.

You can try it by running this code and then reloading the page:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

```js run
window.onbeforeunload = function() {
  return false;
};
```

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
```online
你也可以单击以下 `<iframe>` 中的按钮来设置处理器，然后单击链接：
=======
For historical reasons, returning a non-empty string also counts as canceling the event. Some time ago browsers used show it as a message, but as the [modern specification](https://html.spec.whatwg.org/#unloading-documents) says, they shouldn't.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

Here's an example:

```js run
window.onbeforeunload = function() {
  return "There are unsaved changes. Leave now?";
};
```

The behavior was changed, because some webmasters abused this event handler by showing misleading and annoying messages. So right now old browsers still may show it as a message, but aside of that -- there's no way to customize the message shown to the user.

## readyState

如果在加载文档之后设置 `DOMContentLoaded` 处理器会发生什么？

很自然地，它从未运行过。

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
在某些情况下，我们不确定文档是否已经准备就绪，比如一个具有 `async` 属性的脚本加载并异步运行。取决于网络，它可能在文档完成之前加载和执行，或者在此之后，我们无法确定。因此，我们应该能够知道文件的当前状态。

`document.readyState` 属性为我们提供了一些关于它的信息。有三个可能的值：
=======
There are cases when we are not sure whether the document is ready or not. We'd like our function to execute when the DOM is loaded, be it now or later.

The `document.readyState` property tells us about the current loading state.

There are 3 possible values:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

- `"loading"` —— 文档正在被加载。
- `"interactive"` —— 文档被全部读取。
- `"complete"` —— 文档被全部读取，所有的资源（图像之类的）都被加载。

因此我们检查 `document.readyState` 并设置一个处理器，或在代码准备就绪时立即执行它。

就像这样：

```js
function work() { /*...*/ }

if (document.readyState == 'loading') {
  // loading yet, wait for the event
  document.addEventListener('DOMContentLoaded', work);
} else {
  // DOM is ready!
  work();
}
```

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
有一个 `readystatechange` 事件，当状态发生变化时触发，因此我们可以打印如下所有这些状态：
=======
There's also `readystatechange` event that triggers when the state changes, so we can print all these states like this:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

```js run
// current state
console.log(document.readyState);

// print state changes
document.addEventListener('readystatechange', () => console.log(document.readyState));
```

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
`readystatechange` 事件是跟踪文档加载状态的另一种机制，它很早就存在了。现在则很少被使用，但我们还是需要来讨论一下它的完整性。

`readystatechange` 在其他事件中的地位？

要查看时间，这里有一个带有 `<iframe>`、`<img>` 和记录事件的处理器：
=======
The `readystatechange` event is an alternative mechanics of tracking the document loading state, it appeared long ago. Nowadays, it is rarely used.

Let's see the full events flow for the completeness.

Here's a document with `<iframe>`, `<img>` and handlers that log events:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md

```html
<script>
  log('initial readyState:' + document.readyState);

  document.addEventListener('readystatechange', () => log('readyState:' + document.readyState));
  document.addEventListener('DOMContentLoaded', () => log('DOMContentLoaded'));

  window.onload = () => log('window onload');
</script>

<iframe src="iframe.html" onload="log('iframe onload')"></iframe>

<img src="http://en.js.cx/clipart/train.gif" id="img">
<script>
  img.onload = () => log('img onload');
</script>
```

[在 sandbox](sandbox:readystate) 中的运行示例。

典型输出：
1. [1] initial readyState:loading
2. [2] readyState:interactive
3. [2] DOMContentLoaded
4. [3] iframe onload
5. [4] img onload
6. [4] readyState:complete
7. [4] window onload

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
方括号中的数字表示发生这种情况的大致时间。实际时间会长一些，但标记为相同数字的事件几乎是同时发生的（+- 几毫秒）。

- `document.readyState` 在 `DOMContentLoaded` 之前会立即变成了 `interactive`。这两个事件的意义没有任何差别。
- 当所有资源（`iframe` 和 `img`）都被加载后，`document.readyState` 变成了 `complete`。这里我们可以发现，它大约发生在 `img.onload` (`img` 是最后的资源) 和 `window.onload` 之间。转换到 `complete` 状态的意义与 `window.onload` 一致。区别在于 `window.onload` 在所有其他 `load` 处理器之后一直有效。
=======
The numbers in square brackets denote the approximate time of when it happens. Events labeled with the same digit happen approximately at the same time (+- a few ms).

- `document.readyState` becomes `interactive` right before `DOMContentLoaded`. These two things actually mean the same.
- `document.readyState` becomes `complete` when all resources (`iframe` and `img`) are loaded. Here we can see that it happens in about the same time as `img.onload` (`img` is the last resource) and `window.onload`. Switching to `complete` state means the same as `window.onload`. The difference is that `window.onload` always works after all other `load` handlers.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md


## 总结

<<<<<<< HEAD:2-ui/3-event-details/10-onload-ondomcontentloaded/article.md
页面生命周期事件：

- 当 DOM 准备就绪时，`DOMContentLoaded` 事件就会在 `document` 上触发。在这个阶段，我们可以将 JavaScript 应用于元素。
  - 除了 `async` 或 `defer` 的脚本外，所有的脚本都会被执行。
  - 图像和其他资源仍然可以继续被加载。
- 当页面和所有资源被加载时，`load` 事件会在 `window` 上被触发。我们很少使用它，因为通常没有必要去等待那么久。
- 当用户想要离开页面时，`beforeunload` 事件会在 `window` 上被触发。如果他返回一个字符串，那么浏览器就会以问题的形式向用户确认是否真的要离开。
- 当用户最终离开时，`unload` 事件会在 `window` 上被触发，在处理器中，我们只能做一些简单的事情，不会涉及到延迟或询问用户。正是由于这个限制，它很少被使用。
- `document.readyState` 是文档的当前状态，可以在 `readystatechange` 事件中跟踪变更：
  - `loading` —— 文档正在被加载。
  - `interactive` —— 文档被解析，大概是与 `DOMContentLoaded` 同时发生，而不是在它之前发生。
  - `complete` —— 文档和资源被加载，与 `window.onload` 同时发生，而不是在它之前发生。
=======
Page load events:

- `DOMContentLoaded` event triggers on `document` when DOM is ready. We can apply JavaScript to elements at this stage.
  - Script such as `<script>...</script>` or `<script src="..."></script>` block DOMContentLoaded, the browser waits for them to execute.
  - Images and other resources may also still continue loading.
- `load` event on `window` triggers when the page and all resources are loaded. We rarely use it, because there's usually no need to wait for so long.
- `beforeunload` event on `window` triggers when the user wants to leave the page. If we cancel the event, browser asks whether the user really wants to leave (e.g we have unsaved changes).
- `unload` event on `window` triggers when the user is finally leaving, in the handler we can only do simple things that do not involve delays or asking a user. Because of that limitation, it's rarely used. We can send out a network request with `navigator.sendBeacon`.
- `document.readyState` is the current state of the document, changes can be tracked in the `readystatechange` event:
  - `loading` -- the document is loading.
  - `interactive` -- the document is parsed, happens at about the same time as `DOMContentLoaded`, but before it.
  - `complete` -- the document and resources are loaded, happens at about the same time as `window.onload`, but before it.
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:2-ui/5-loading/01-onload-ondomcontentloaded/article.md