---
title: Управление состоянием (State management)
---

Компоненты Svelte имеют встроенное управление состоянием с помощью методов `get` и `set`. Но когда размер вашего приложение выходит за определенные рамки, вы можете обнаружить, что передача данных между компонентами становится трудоемкой.
<!-- Svelte components have built-in state management via the `get` and `set` methods. But as your application grows beyond a certain size, you may find that passing data between components becomes laborious. -->

Например, у вас может быть компонент `<Options>` внутри компонента `<Sidebar>`, который позволяет пользователю управлять поведением компонента `<MainView>`. Вы можете использовать привязки или события для 'отправки' информации из `<Options>` через `<Sidebar>` на общего предка - например, `<App>` - который затем должен отправить его на `<MainView>`. Но это громоздко, особенно если вы решите, что хотите разбить `<MainView>` на несколько компонентов.
<!-- For example, you might have an `<Options>` component inside a `<Sidebar>` component that allows the user to control the behaviour of a `<MainView>` component. You could use bindings or events to 'send' information up from `<Options>` through `<Sidebar>` to a common ancestor — say `<App>` — which would then have the responsibility of sending it back down to `<MainView>`. But that's cumbersome, especially if you decide you want to break `<MainView>` up into a set of smaller components. -->

Хорошим решением этой проблемы является использование *глобального хранилища* данных, которое пересекает вашу иерархию компонентов. React имеет [Redux](https://redux.js.org/) и [MobX](https://mobx.js.org/index.html) (хотя эти библиотеки можно использовать где угодно, в том числе с Svelte), а Vue имеет [Vuex](https://vuex.vuejs.org/en/).
<!-- Instead, a popular solution to this problem is to use a *global store* of data that cuts across your component hierarchy. React has [Redux](https://redux.js.org/) and [MobX](https://mobx.js.org/index.html) (though these libraries can be used anywhere, including with Svelte), and Vue has [Vuex](https://vuex.vuejs.org/en/). -->

У Svelte есть `Store`. `Store` можно использовать в любом приложении JavaScript, но особенно хорошо он подходит для приложений Svelte.
<!-- Svelte has `Store`. `Store` can be used in any JavaScript app, but it's particularly well-suited to Svelte apps. -->


### Основа (The basics)

Импортируйте `Store` из `svelte/store.js` (не забудьте включить фигурные скобки, так как это *named import*), а затем создайте новое хранилище с некоторыми (необязательными) данными:
<!-- Import `Store` from `svelte/store.js` (remember to include the curly braces, as it's a *named import*), then create a new store with some (optional) data: -->

```js
import { Store } from 'svelte/store.js';

const store = new Store({
	name: 'world'
});
```

Каждый экземпляр `Store` имеет методы `get`, `set`, `on` и `fire`, которые работают одинаково с их копиями в компоненте Svelte:
<!-- Each instance of `Store` has `get`, `set`, `on` and `fire` methods that behave identically to their counterparts on a Svelte component: -->

```js
const { name } = store.get(); // 'world'

store.on('state', ({ current }) => {
	console.log(`hello ${current.name}`);
});

store.set({ name: 'everybody' }); // 'hello everybody'
```



### Создание компонентов с хранилищами (Creating components with stores)

Давайте адаптируем наш [самый первый пример](guide#understanding-svelte-components):
<!-- Let's adapt our [very first example](guide#understanding-svelte-components): -->

```html
<!-- { repl: false } -->
<h1>Hello {$name}!</h1>
<Greeting/>

<script>
	import Greeting from './Greeting.html';

	export default {
		components: { Greeting }
	};
</script>
```

```html
<!--{ filename: 'Greeting.html' }-->
<p>It's nice to see you, {$name}</p>
```

```js
/* { filename: 'main.js' } */
import App from './App.html';
import { Store } from 'svelte/store.js';

const store = new Store({
	name: 'world'
});

const app = new App({
	target: document.querySelector('main'),
	store
});

window.store = store; // useful for debugging!
```

Есть три важных момента:
<!-- There are three important things to notice: -->

* Мы передаем `store` в `new App(...)` вместо `data`
<!-- We're passing `store` to `new App(...)` instead of `data` -->
* Шаблон ссылается на `$name` вместо `name`. Префикс `$` говорит Svelte, что `name` является *свойством хранилища (store property)*
<!-- The template refers to `$name` instead of `name`. The `$` prefix tells Svelte that `name` is a *store property* -->
* Поскольку `<Greeting>` является дочерним элементом `<App>`, он также имеет доступ к хранилищу. Без него `<App>` должен передать свойство `name` в качестве свойства компонента (`<Greeting name={name}/>`)
<!-- Because `<Greeting>` is a child of `<App>`, it also has access to the store. Without it, `<App>` would have to pass the `name` property down as a component property (`<Greeting name={name}/>`) -->

Компоненты, зависящие от свойств хранилища, будут повторно рендериться при каждом изменении.
<!-- Components that depend on store properties will re-render whenever they change. -->


### Декларативные хранилища (Declarative stores)

В качестве альтернативы добавлению опции `store` при создании экземпляра сам компонент может объявлять зависимость от хранилища:
<!-- As an alternative to adding the `store` option when instantiating, the component itself can declare a dependency on a store: -->

```html
<!-- { title: 'Declarative stores' } -->
<h1>Hello {$name}!</h1>
<Greeting/>

<script>
	import Greeting from './Greeting.html';
	import store from './store.js';

	export default {
		store: () => store,
		components: { Greeting }
	};
</script>
```

```html
<!--{ filename: 'Greeting.html' }-->
<p>It's nice to see you, {$name}</p>
```

```js
/* { filename: 'store.js' } */
import { Store } from 'svelte/store.js';
export default new Store({ name: 'world' });
```

Обратите внимание, что опция `store` — это функция, которая *возвращает* хранилище, а не само хранилище, что обеспечивает большую гибкость.
<!-- Note that the `store` option is a function that *returns* a store, rather than the store itself — this provides greater flexibility. -->


### Вычисляемые свойства хранилища (Computed store properties)

Подобно компонентам, хранилища могут иметь вычисляемые свойства:
<!-- Just like components, stores can have computed properties: -->

```js
store = new Store({
	width: 10,
	height: 10,
	depth: 10,
	density: 3
});

store.compute(
	'volume',
	['width', 'height', 'depth'],
	(width, height, depth) => width * height * depth
);

store.get().volume; // 1000

store.set({ width: 20 });
store.get().volume; // 2000

store.compute(
	'mass',
	['volume', 'density'],
	(volume, density) => volume * density
);

store.get().mass; // 6000
```

Первый аргумент — это имя вычисленного свойства. Второй — это массив *зависимостей* — это могут быть свойства данных или другие рассчитанные свойства. Третий аргумент — это функция, которая пересчитывает значение всякий раз, когда изменяется зависимость.
<!-- The first argument is the name of the computed property. The second is an array of *dependencies* — these can be data properties or other computed properties. The third argument is a function that recomputes the value whenever the dependencies change. -->

Компонент, который был подключен к этому хранилищу, может ссылаться на `{$volume}` и `{$mass}`, как и на любое другое свойство хранилища.
<!-- A component that was connected to this store could reference `{$volume}` and `{$mass}`, just like any other store property. -->


### Доступ к внутренним компонентам хранилища (Accessing the store inside components)

Каждый компонент получает ссылку на `this.store`. Это позволяет прикрепить поведение в `oncreate`...
<!-- Each component gets a reference to `this.store`. This allows you to attach behaviours in `oncreate`... -->

```html
<!-- { repl: false } -->
<script>
	export default {
		oncreate() {
			const listener = this.store.on('state', ({ current }) => {
				// ...
			});
	
			// listeners are not automatically removed — cancel
			// them to prevent memory leaks
			this.on('destroy', listener.cancel);
		}
	};
</script>
```
... или вызывать методы хранилища в обработчиках событий, используя тот же префикс `$` как свойства данных:
<!-- ...or call store methods in your event handlers, using the same `$` prefix as data properties: -->

```html
<!-- { repl: false } -->
<button on:click="$set({ muted: true })">
	Mute audio
</button>
```


### Пользовательские методы хранилища (Custom store methods)

В `Store` не имеет понятий *действия* или *коммиты*, как в Redux и Vuex. Вместо этого состояние всегда обновляется с помощью `store.set(...)`.
<!-- `Store` doesn't have a concept of *actions* or *commits*, like Redux and Vuex. Instead, state is always updated with `store.set(...)`. -->

Вы можете реализовать пользовательскую логику путем подклассификации `Store`:
<!-- You can implement custom logic by subclassing `Store`: -->

```js
class TodoStore extends Store {
	addTodo(description) {
		const todo = {
			id: generateUniqueId(),
			done: false,
			description
		};

		const todos = this.get().todos.concat(todo);
		this.set({ todos });
	}

	toggleTodo(id) {
		const todos = this.get().todos.map(todo => {
			if (todo.id === id) {
				return {
					id,
					done: !todo.done,
					description: todo.description
				};
			}

			return todo;
		});

		this.set({ todos });
	}
}

const store = new TodoStore({
	todos: []
});

store.addTodo('Finish writing this documentation');
```

Методы могут обновлять хранилище асинхронно:
<!-- Methods can update the store asynchronously: -->

```js
class NasdaqTracker extends Store {
	async fetchStockPrices(ticker) {
		const token = this.token = {};
		const prices = await fetch(`/api/prices/${ticker}`).then(r => r.json());
		if (token !== this.token) return; // invalidated by subsequent request

		this.set({ prices });
	}
}

const store = new NasdaqTracker();
store.fetchStockPrices('AMZN');
```

Вы можете вызвать эти методы в своих компонентах, как и встроенные методы:
<!-- You can call these methods in your components, just like the built-in methods: -->


```html
<!-- { repl: false } -->
<input
	placeholder="Enter a stock ticker"
	on:change="$fetchStockPrices(this.value)"
>
```

### Привязка хранилища (Store bindings)

Вы можете привязывать значения в хранилище так же, как вы привязываете значения компонентов — просто добавьте префикс `$`:
<!-- You can bind to store values just like you bind to component values — just add the `$` prefix: -->

```html
<!-- { repl: false } -->
<!-- global audio volume control -->
<input bind:value=$volume type=range min=0 max=1 step=0.01>
```


### Использование свойств хранилища в вычисляемых свойствах (Using store properties in computed properties)

Как и в шаблонах, вы можете получить доступ к свойствам хранилища в вычисляемых свойствах компонентов — с помощью префикса `$`:
<!-- Just as in templates, you can access store properties in component computed properties by prefixing them with `$`: -->

```html
<!-- { repl: false } -->
{#if isVisible}
	<div class="todo {todo.done ? 'done': ''}">
		{todo.description}
	</div>
{/if}

<script>
	export default {
		computed: {
			// `todo` is a component property, `$filter` is
			// a store property
			isVisible: ({ todo, $filter }) => {
				if ($filter === 'all') return true;
				if ($filter === 'done') return todo.done;
				if ($filter === 'pending') return !todo.done;
			}
		}
	};
</script>
```


### Встроенная оптимизация (Built-in optimisations)

Компилятор Svelte знает, какие свойства хранилища нужны вашим компонентам (благодаря префиксу `$`), и записывает код, который прослушивает изменения этих свойств. Поэтому вам не нужно беспокоиться о том, что у вас много свойств в хранилище, даже тех, которые часто обновляются — компоненты, которые их не используют, не будут затронуты.
<!-- The Svelte compiler knows which store properties your components are interested in (because of the `$` prefix), and writes code that only listens for changes to those properties. Because of that, you needn't worry about having many properties on your store, even ones that update frequently — components that don't use them will be unaffected. -->

