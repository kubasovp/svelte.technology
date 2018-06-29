---
title: Рендеринг на стороне сервера (Server-side rendering, SSR)
---

До сих пор мы обсуждали создание компонентов Svelte *на клиенте*, которым является браузер. Но вы также можете рендерить компоненты Svelte в Node.js. Это может привести к повышению производительности, поскольку это означает, что приложение начинает рендеринг, пока страница все еще загружается, прежде чем выполняется какой-либо JavaScript. Также, зачастую это дает преимущества в SEO и может быть полезено для людей, работающих с более старыми браузерами, которые не могут или не хотят запускать ваш JavaScript по какой-либо причине.
<!-- So far, we've discussed creating Svelte components *on the client*, which is to say the browser. But you can also render Svelte components in Node.js. This can result in better perceived performance as it means the application starts rendering while the page is still downloading, before any JavaScript executes. It also has SEO advantages in some cases, and can be beneficial for people running older browsers that can't or won't run your JavaScript for whatever reason. -->


### Использование компилятора (Using the compiler)

Если вы используете компилятор Svelte напрямую или с помощью интеграции, как, например [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) или [svelte-loader](https://github.com/sveltejs/svelte-loader), вы можете указать ему создать серверный компонент, передав опцию `generate: 'ssr'`:
<!-- If you're using the Svelte compiler, whether directly or via an integration like [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) or [svelte-loader](https://github.com/sveltejs/svelte-loader), you can tell it to generate a server-side component by passing the `generate: 'ssr'` option: -->

```js
const { js } = svelte.compile(source, {
	generate: 'ssr' // as opposed to 'dom', the default
});
```


### Регистрация Svelte (Registering Svelte)

В качестве альтернативы, простой способ использовать серверный рендеринг — *зарегистрировать* его:
<!-- Alternatively, an easy way to use the server-side renderer is to *register* it: -->

```js
require('svelte/ssr/register');
```

Теперь вы можете `потребовать` (`require`) свои компоненты:
<!-- Now you can `require` your components: -->

```js
const Thing = require('./components/Thing.html');
```

Если вы предпочитаете использовать другое расширение файла, вы можете передать такие параметры:
<!-- If you prefer to use a different file extension, you can pass options like so: -->

```js
require('svelte/ssr/register')({
	extensions: ['.svelte']
});
```


### Server-side API

Компоненты имеют другой API в Node.js — вместо того, чтобы создавать экземпляры с методами `set(...)` и `get()`, компонент представляет собой объект с методом `render(data, options)`:
<!-- Components have a different API in Node.js – rather than creating instances with `set(...)` and `get()` methods, a component is an object with a `render(data, options)` method: -->

```js
require('svelte/ssr/register');
const Thing = require('./components/Thing.html');

const data = { answer: 42 };
const { html, css, head } = Thing.render(data);
```

Все ваши [default data](guide#default-data), [computed properties](guide#computed-properties), [helpers](guide#helpers) и [nested components](guide#nested-components) будут работать как ожидалось.
<!-- All your [default data](guide#default-data), [computed properties](guide#computed-properties), [helpers](guide#helpers) and [nested components](guide#nested-components) will work as expected. -->

Любые функции `oncreate` или методы компонентов *не* будут выполняться — они применяются только к клиентским компонентам.
<!-- Any `oncreate` functions or component methods will *not* run — these only apply to client-side components. -->

> Компилятор SSR будет генерировать модуль CommonJS для каждого из ваших компонентов, что означает, что операторы `import` и `export` преобразуются в их эквиваленты `require` и `module.exports`. Если ваши компоненты имеют некомпонентные зависимости, они также должны работать как модули CommonJS в Node. Если вы используете модули ES2015, мы рекомендуем модуль [`esm`](https://github.com/standard-things/esm) для автоматического преобразования их в CommonJS.
<!-- The SSR compiler will generate a CommonJS module for each of your components – meaning that `import` and `export` statements are converted into their `require` and `module.exports` equivalents. If your components have non-component dependencies, they must also work as CommonJS modules in Node. If you're using ES2015 modules, we recommend the [`esm`](https://github.com/standard-things/esm) module for automatically converting them to CommonJS. -->



#### Использование хранилища данных (Using stores)

Если ваши компоненты используют [stores](guide#state-management), используйте второй аргумент:
<!-- If your components use [stores](guide#state-management), use the second argument: -->

```js
const { Store } = require('svelte/store.umd.js');

const { html } = Thing.render(data, {
	store: new Store({
		foo: 'bar'
	})
});
```


#### Рендеринг стилей (Rendering styles)

Вы можете извлекать любые [scoped styles](guide#scoped-styles), которые используются компонентом или его дочерними элементами:
<!-- You can also extract any [scoped styles](guide#scoped-styles) that are used by the component or its children: -->

```js
const { css } = Thing.render(data);
```

Вы можете поместить полученный `css` в отдельную таблицу стилей или включить их на странице внутри тега `<style>`. Если вы это сделаете, вы, вероятно, захотите, чтобы компилятор на стороне клиента снова включил CSS. Для CLI используйте флаг `--no-css`. В инструментах сборки, таких как `rollup-plugin-svelte`, передайте опцию `css: false`.
<!-- You could put the resulting `css` in a separate stylesheet, or include them in the page inside a `<style>` tag. If you do this, you will probably want to prevent the client-side compiler from including the CSS again. For the CLI, use the `--no-css` flag. In build tool integrations like `rollup-plugin-svelte`, pass the `css: false` option. -->



#### Рендеринг содержимого тега `<head>` (Rendering `<head>` contents)

Если ваш компонент, или любой из его дочерних элементов, использует `<svelte:head>` [component](guide#-head-tags), вы можете извлечь содержимое:
<!-- If your component, any of its children, use the `<svelte:head>` [component](guide#-head-tags), you can extract the contents: -->

```js
const { head } = Thing.render(data);
```
