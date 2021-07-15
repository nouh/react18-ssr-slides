---
# try also 'default' to start simple
theme: Seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: /erik-mclean-za3ADPq8mpo-unsplash.jpg
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki

fonts:
  sans: "Roboto"
  serif: "Roboto Slab"
  mono: "Fira Code"
# some information about the slides, markdown enabled
info: |
  React 18 SSR
---

# React 18 给 SSR 带来的新变化

### By Zach Zhao

<!-- <div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
</div> -->

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# SSR 基本原理

<br/>
<br/>

<div class="flex-row flex items-center mb-10">
<p class="mr-2 w-30">Without SSR:</p>
<img src='/blank.png' class="w-50 mr-10">
Rendered<mdi:arrow-right class="text-4xl text-orange-400"/>
<img src='/full.png' class="w-50 ml-10">
</div>

<div class="flex-row flex items-center">
<p class="mr-2 w-30">With SSR:</p>
<img src='/skeleton.png' class="w-50 mr-10">
Hydrated<mdi:arrow-right class="text-4xl text-orange-400"/>
<img src='/full.png' class="w-50 ml-10">
</div>

---

# SSR 阶段分解

<img src='/ssr.png' class="w-80%">

<!-- fetch data (server) → render to HTML (server) → load code (client) → hydrate (client) -->

- On the server, fetch data for the <kbd>entire</kbd> app.
- Then, on the server, render the <kbd>entire</kbd> app to HTML and send it in the response.
- Then, on the client, load the JavaScript code for the <kbd>entire</kbd> app.
- Then, on the client, connect the JavaScript logic to the server-generated HTML for the <kbd>entire</kbd> app (this is “hydration”).

---

# SSR 现存问题

- You have to fetch everything before you can show anything

- You have to load everything before you can hydrate anything

- You have to hydrate everything before you can interact with anything

---

# React 18 Alpha 对 SSR 的改进

- Streaming HTML (on the server)

  > To opt into it, you’ll need to switch from <kbd>renderToString</kbd> to the new <kbd>pipeToNodeWritable</kbd> method

- Selective Hydration (on the client)
  > To opt into it, you’ll need to switch from <kbd>ReactDOM.render</kbd> to the new <kbd>ReactDOM.createRoot</kbd> on the client and then start wrapping parts of your app with `<suspense>`

---

# API 的变化

> 以前 React 没有对 Suspense 的服务端支持，但在 React18 带来了一些改变，并且不同的 API 对其的支持程度有所不同

<br/>

## ReactDOMServer

- 📝 renderToString: 仍有效 (对 Suspense 提供有限支持).
- 🧑‍💻 renderToNodeStream: 已废弃 (支持 Suspense，不支持 Streaming).
- 📤 pipeToNodeWritable: 推荐使用 (完整支持 Suspense 和 Streaming).

## ReactDOM

- 👏 render: 仍有效 (不支持 concurrent mode)
- 🚀 createRoot: 推荐使用 (支持 concurrent mode)

<br/>

---


```js {all|6-10|all}
import * as ReactDOM from "react-dom";
import App from "App";

const container = document.getElementById("app");

// Initial render.
ReactDOM.render(<App tab="home" />, container);

// During an update, React would access the root of the DOM element.
ReactDOM.render(<App tab="profile" />, container);
```
<div class="flex flex-col items-center">
<mdi:arrow-down class="text-3xl text-orange-400"/>
</div>

```js {all|7|8-13}
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Create a root.
const root = ReactDOM.createRoot(container);

// Initial render: Render an element to the root.
root.render(<App tab="home" />);

// During an update, there's no need to pass the container again.
root.render(<App tab="profile" />);
```


--- 

# Streaming HTML

<div class="flex flex-row space-x-8">

<div>
```html {all|6-8|all}
<Layout>
  <NavBar />
  <Sidebar />
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

```html {all|12|all}
<main>
  <nav>
    <a href="/">Home</a>
  </nav>
  <aside>
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <p>Hello world</p>
  </article>
  <section id="comments-spinner">
    <img width="400" src="spinner.gif" alt="Loading..." />
  </section>
</main>
```

</div>

<div class="flex items-center">
<img src="/ssr_loading.png" class="w-120">
</div>
</div>

---

<div class="flex flex-row space-x-8">
<div class="flex items-center">

```html {all|6-10|all}
<div hidden id="comments">
  <!-- Comments -->
  <p>First comment</p>
  <p>Second comment</p>
</div>
<script>
  document
    .getElementById("sections-spinner")
    .replaceChildren(document.getElementById("comments"));
</script>
```

</div>

<div class="flex items-center">
  <img src="/skeleton.png" class="w-120">
</div>

</div>

---

# Selective Hydration

<div class="flex flex-col space-y-4">
<div class="flex items-center">

```js {all|3|all}
import { lazy } from "react";

const Comments = lazy(() => import("./Comments.js"));

// ...

<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>;
```

</div>

<div class="flex flex-row space-x-4 items-center">
  <img src="/skeleton.png" class="w-60">
  <mdi:arrow-right class="text-3xl text-orange-400"/>
  <img src="/partial.png" class="w-60">
  <mdi:arrow-right  class="text-3xl text-orange-400"/>
  <img src="/full.png" class="w-60">
</div>

</div>

---

# Hydrating Can Be Interrupted

  > 如果一个组件被Suspense包裹，那么它的注水过程不再阻塞浏览器响应其他的行为

<br/>
<img src="/hydrating.png" class="w-100">

---

# Hydrating Can Be Prioritized

  > React 可以先记录下评论区的点击事件，待评论区被优先注水以后重放这个点击事件，然后再转头回去对侧边栏进行注水

<br/>
<div class="flex flex-row space-x-4 items-center">
  <img src="/click_comments.png" class="w-60">
  <mdi:arrow-right class="text-3xl text-orange-400"/>
  <img src="/hydrating_comments.png" class="w-60">
  <mdi:arrow-right  class="text-3xl text-orange-400"/>
  <img src="/hydrating_sidebar_again.png" class="w-60">
</div>

---

# Conclusion

### React 18 对SSR添加了两个主要功能
<br/>

* Streaming HTML
* Selective Hydration

### 解决了三个长久的SSR问题
<br/>

* You no longer have to wait for all the data to load on the server before sending HTML
* You no longer have to wait for all JavaScript to load to start hydrating
* You no longer have to wait for all components to hydrate to start interacting with the page
