---
title: Специальные элементы (Special elements)
---

В Svelte есть несколько встроенных элементов со специальным поведением.
<!-- Svelte includes a handful of built-in elements with special behaviour. -->


### `<svelte:self>`

Иногда нужно встроить компонент рекурсивно - например, если у вас есть древовидная структура данных. В Svelte это выполняется с помощью тега `<svelte:self>`:
<!-- Sometimes, a component needs to embed itself recursively — for example if you have a tree-like data structure. In Svelte, that's accomplished with the `<svelte:self>` tag: -->

```html
<!-- { title: '<svelte:self> tags' } -->
{#if countdown > 0}
	<p>{countdown}</p>
	<svelte:self countdown="{countdown - 1}"/>
{:else}
	<p>liftoff!</p>
{/if}
```

```json
/* { hidden: true } */
{
	countdown: 5
}
```


### `<svelte:component>`

Если вы не знаете, какой компонент должен отображать до запуска приложения, другими словами, он управляется по состоянию - вы можете использовать `<svelte:component>`:
<!-- If you don't know what kind of component to render until the app runs — in other words, it's driven by state — you can use `<svelte:component>`: -->

```html
<!-- { title: '<svelte:component> tags' } -->
<input type=checkbox bind:checked=foo> foo
<svelte:component this="{foo ? Red : Blue}" name="thing"/>

<script>
	import Red from './Red.html';
	import Blue from './Blue.html';

	export default {
		data() {
			return { Red, Blue }
		}
	};
</script>
```

```html
<!--{ hidden: true, filename: 'Red.html' }-->
<p style="color: red">Red {name}</p>
```

```html
<!--{ hidden: true, filename: 'Blue.html' }-->
<p style="color: blue">Blue {name}</p>
```

> Обратите внимание, что `Red` и `Blue` являются элементами `data`, а *не* `components`, в отличие от `<Red>` или `<Blue>`.
<!-- Note that `Red` and `Blue` are items in `data`, *not* `components`, unlike if we were doing `<Red>` or `<Blue>`. -->

Выражение внутри `this="{...}"` может быть любым допустимым выражением JavaScript. Например, это может быть [вычисляемое свойство](guide#computed-properties): 
<!-- The expression inside the `this="{...}"` can be any valid JavaScript expression. For example, it could be a [computed property](guide#computed-properties): -->

```html
<!-- { title: '<svelte:component> with computed' } -->
<label><input type=radio bind:group=size value=small> small</label>
<label><input type=radio bind:group=size value=medium> medium</label>
<label><input type=radio bind:group=size value=large> large</label>

<svelte:component this={Size}/>

<script>
	import Small from './Small.html';
	import Medium from './Medium.html';
	import Large from './Large.html';

	export default {
		computed: {
			Size: ({size}) => {
				if (size === 'small') return Small;
				if (size === 'medium') return Medium;
				return Large;
			}
		}
	};
</script>
```

```html
<!--{ filename: 'Small.html' }-->
<p style="font-size: 12px">small</p>
```

```html
<!--{ filename: 'Medium.html' }-->
<p style="font-size: 18px">medium</p>
```

```html
<!--{ filename: 'Large.html' }-->
<p style="font-size: 32px">LARGE</p>
```

```json
/* { hidden: true } */
{
	size: "medium"
}
```


### `<svelte:window>`

Тег `<svelte:window>` дает вам удобный способ декларативно добавлять обработчики событий в `window`. Обработчики автоматически удаляются при уничтожении компонента.
<!-- The `<svelte:window>` tag gives you a convenient way to declaratively add event listeners to `window`. Event listeners are automatically removed when the component is destroyed. -->

```html
<!-- { title: '<svelte:window> tags' } -->
<svelte:window on:keydown="set({ key: event.key, keyCode: event.keyCode })"/>

{#if key}
	<p><kbd>{key === ' ' ? 'Space' : key}</kbd> (code {keyCode})</p>
{:else}
	<p>click in this window and press any key</p>
{/if}

<style>
	kbd {
		background-color: #eee;
		border: 2px solid #f4f4f4;
		border-right-color: #ddd;
		border-bottom-color: #ddd;
		font-size: 2em;
		margin: 0 0.5em 0 0;
		padding: 0.5em 0.8em;
		font-family: Inconsolata;
	}
</style>
```

Вы можете привязать к определенным значениям - по-прежнему `innerWidth`, `outerWidth`, `innerHeight`, `outerHeight`, `scrollX`, `scrollY` и `online`:
<!-- You can also bind to certain values — so far `innerWidth`, `outerWidth`, `innerHeight`, `outerHeight`, `scrollX`, `scrollY` and `online`: -->

```html
<!-- { title: '<svelte:window> bindings' } -->
<svelte:window bind:scrollY=y/>

<div class="background"></div>
<p class="fixed">user has scrolled {y} pixels</p>

<style>
	.background {
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 9999px;
		background: linear-gradient(to bottom, #7db9e8 0%,#0a1d33 100%);
	}

	.fixed {
		position: fixed;
		top: 1em;
		left: 1em;
		color: white;
	}
</style>
```


### `<svelte:head>`

Если вы создаете приложение с Svelte — особенно если вы используете [Sapper](https://sapper.svelte.technology) - скорее всего, вам нужно будет добавить некоторый контент в `<head>` вашей страницы, например, добавить элемент `<title>`.
<!-- If you're building an application with Svelte — particularly if you're using [Sapper](https://sapper.svelte.technology) — then it's likely you'll need to add some content to the `<head>` of your page, such as adding a `<title>` element. -->

Вы можете сделать это с помощью тега `<svelte:head>`:
<!-- You can do that with the `<svelte:head>` tag: -->

```html
<!-- { title: '<svelte:head> tags' } -->
<svelte:head>
	<title>{post.title} • My blog</title>
</svelte:head>
```

При [серверном рендеринге](guide#server-side-rendering) содержимое `<head>` может быть выделено отдельно от остальной части разметки.
<!-- When [server rendering](guide#server-side-rendering), the `<head>` contents can be extracted separately to the rest of the markup. -->