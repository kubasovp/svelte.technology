---
title: Продвинутые (Advanced)
---


### Ключи для каждого блока (Keyed each blocks)

Связывание *ключа* с блоком позволяет Svelte быть более умным в добавлении и удалении элементов списка. Для этого добавьте `(expression)`, которое однозначно идентифицирует каждый элемент списка:
<!-- Associating a *key* with a block allows Svelte to be smarter about how it adds and removes items to and from a list. To do so, add an `(expression)` that uniquely identifies each member of the list: -->

```html
<!-- { repl: false } -->
{#each people as person (person.name)}
	<div>{person.name}</div>
{/each}
```

Легче показать этот эффект, чем описать его. Откройте в REPL следующий пример:
<!-- It's easier to show the effect of this than to describe it. Open the following example in the REPL: -->

```html
<!-- { title: 'Keyed each blocks' } -->
<button on:click="update()">update</button>

<section>
	<h2>Keyed</h2>
	{#each people as person (person.name)}
		<div transition:slide>{person.name}</div>
	{/each}
</section>

<section>
	<h2>Non-keyed</h2>
	{#each people as person}
		<div transition:slide>{person.name}</div>
	{/each}
</section>

<style>
	button {
		display: block;
	}

	section {
		width: 10em;
		float: left;
	}
</style>

<script>
	import { slide } from 'svelte-transitions';

	var people = ['Alice', 'Barry', 'Cecilia', 'Douglas', 'Eleanor', 'Felix', 'Grace', 'Horatio', 'Isabelle'];

	function random() {
		return people
			.filter(() => Math.random() < 0.5)
			.map(name => ({ name }))
	}

	export default {
		data() {
			return { people: random() };
		},

		methods: {
			update() {
				this.set({ people: random() });
			}
		},

		transitions: { slide }
	};
</script>
```


### Hydration

Если вы используете [рендеринг на стороне сервера](guide#server-side-rendering), вероятно, вам нужно будет создать клиентскую версию вашего приложения *поверх* серверной. Нативный способ сделать это включает удаление всего существующего DOM и рендеринга приложения на стороне клиента:
<!-- If you're using [server-side rendering](guide#server-side-rendering), it's likely that you'll need to create a client-side version of your app *on top of* the server-rendered version. A naive way to do that would involve removing all the existing DOM and rendering the client-side app in its place: -->

```js
import App from './App.html';

const target = document.querySelector('#element-with-server-rendered-html');

// avoid doing this!
target.innerHTML = '';
new App({
	target
});
```

В идеале мы хотим повторно использовать существующий DOM. Этот процесс называется *hydration*. Во-первых, нам нужно сообщить компилятору включить код, необходимый для работы *hydration*, передав опцию `hydratable: true`:
<!-- Ideally, we want to reuse the existing DOM instead. This process is called *hydration*. First, we need to tell the compiler to include the code necessary for hydration to work by passing the `hydratable: true` option: -->

```js
const { js } = svelte.compile(source, {
	hydratable: true
});
```

(Скорее всего, вы передадите эту опцию в [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) или [svelte-loader](https://github.com/sveltejs/svelte-loader).)
<!-- (Most likely, you'll be passing this option to [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) or [svelte-loader](https://github.com/sveltejs/svelte-loader).) -->

Затем, когда мы создаем экземпляр клиентского компонента, мы говорим ему использовать существующую DOM с помощью `hydrate: true`:
<!-- Then, when we instantiate the client-side component, we tell it to use the existing DOM with `hydrate: true`: -->

```js
import App from './App.html';

const target = document.querySelector('#element-with-server-rendered-html');

new App({
	target,
	hydrate: true
});
```

> Неважно, если приложение на стороне клиента не в точности соответствует серверному HTML — Svelte будет исправлять DOM по мере его появления.
<!-- It doesn't matter if the client-side app doesn't perfectly match the server-rendered HTML — Svelte will repair the DOM as it goes. -->


### Иммутабельность (Immutable)

Поскольку массивы и объекты *изменяемы*, Svelte должен с осторожностью принимать решения о том, следует ли обновлять вещи, которые относятся к ним.
<!-- Because arrays and objects are *mutable*, Svelte must err on the side of caution when deciding whether or not to update things that refer to them. -->

Но если все ваши данные [иммутабельны](https://ru.wikipedia.org/wiki/Неизменяемый_объект), вы можете использовать опцию `{ immutable: true }` в компиляторе для использования строгого сравнения объектов (используя `===`) во всем приложении. Если у вас есть один компонент, который использует неизменяемые данные, вы можете установить использование строгого сравнения только для этого компонента.
<!-- But if all your data is [immutable](https://en.wikipedia.org/wiki/Immutable_object), you can use the `{ immutable: true }` compiler option to use strict object comparison (using `===`) everywhere in your app. If you have one component that uses immutable data you can set it to use the strict comparison for just that component. -->

В приведенном ниже примере, `searchResults` обычно пересчитывается всякий раз, когда `items` *могут* быть изменены, но с `immutable: true` он будет обновляться только тогда, когда `items` *точно* изменились. Это может улучшить производительность вашего приложения.
<!-- In the example below, `searchResults` would normally be recalculated whenever `items` *might* have changed, but with `immutable: true` it will only update when `items` has *definitely* changed. This can improve the performance of your app. -->

```html
<!-- { repl: false } -->
{#each searchResults as item}
	<div>{item.name}</div>
{/each}

<script>
	import FuzzySearch from 'fuzzy-search';

	export default {
		immutable: true,

		computed: {
			searchResults: ({ searchString, items }) => {
				if (!searchString) return items;

				const searcher = new FuzzySearch(items, ['name', 'location']);
				return searcher.search(searchString);
			}
		}
	}
</script>
```
[Вот пример](repl?demo=immutable), показывающий эффект `immutable: true`.
<!-- [Here's a live example](repl?demo=immutable) showing the effect of `immutable: true`. -->
