---
title: Директивы
---

Директивы это инструкции для элемента или компонента. Они выглядят как атрибуты с символом `:`.
<!-- Directives are element or component-level instructions to Svelte. They look like attributes, except with a `:` character. -->

### Обработчики событий (Event handlers)

В большинстве приложений вам нужно ответить на действия пользователя. В Svelte это делается с помощью директивы `on:[event]`.
<!-- In most applications, you'll need to respond to the user's actions. In Svelte, this is done with the `on:[event]` directive. -->

```html
<!-- { title: 'Event handlers' } -->
<p>Count: {count}</p>
<button on:click="set({ count: count + 1 })">+1</button>
```

```json
/* { hidden: true } */
{
	count: 0
}
```

Когда пользователь кликает по кнопке, Svelte вызывает `component.set (...)` с заданными аргументами. Вы можете вызывать любой метод, принадлежащий этому компоненту (неважно, [встроенный](guide#component-api) или [пользовательский](guide#custom-methods)) и любое свойство (или вычисленное свойство), которое находится в области видимости:
<!-- When the user clicks the button, Svelte calls `component.set(...)` with the provided arguments. You can call any method belonging to the component (whether [built-in](guide#component-api) or [custom](guide#custom-methods)), and any data property (or computed property) that's in scope: -->

```html
<!-- { title: 'Calling custom methods' } -->
<p>Select a category:</p>

{#each categories as category}
	<button on:click="select(category)">select {category}</button>
{/each}

<script>
	export default {
		data() {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		},

		methods: {
			select(name) {
				alert(`selected ${name}`); // серьезно, пожалуйста, не делайте этого
			}
		}
	};
</script>
```

Также, можно получить доступ к объекту события (`event`) при вызове метода:
<!-- You can also access the `event` object in the method call: -->

```html
<!-- { title: 'Accessing `event`' } -->
<div on:mousemove="set({ x: event.clientX, y: event.clientY })">
	coords: {x},{y}
</div>

<style>
	div {
		border: 1px solid purple;
		width: 100%;
		height: 100%;
	}
</style>
```

На нужный узел можно ссылаться с помощью `this`, то есть вы можете делать так:
<!-- The target node can be referenced as `this`, meaning you can do this sort of thing: -->

```html
<!-- { title: 'Calling node methods' } -->
<input on:focus="this.select()" value="click to select">
```


### Пользовательские события (Custom events)

Вы можете определить свои собственные события для обработки сложных пользовательских взаимодействий, таких как перетаскивание, пролистывание (свайп). Более подробную информацию см. в разделе [обработчики пользовательских событий](guide#custom-event-handlers).
<!-- You can define your own custom events to handle complex user interactions like dragging and swiping. See the earlier section on [custom event handlers](guide#custom-event-handlers) for more information. -->


### События в компонентах (Component events)

События — отличный способ для [вложенных компонентов](guide#nested-components) общаться со своими родителями. Давайте перейдем к нашему предыдущему примеру, но превратим его в компонент `<CategoryChooser>`:
<!-- Events are an excellent way for [nested components](guide#nested-components) to communicate with their parents. Let's revisit our earlier example, but turn it into a `<CategoryChooser>` component: -->

```html
<!-- { repl: false } -->
<p>Select a category:</p>

{#each categories as category}
	<button on:click="fire('select', { category })">select {category}</button>
{/each}

<script>
	export default {
		data() {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		}
	};
</script>
```

Когда пользователь нажимает кнопку, компонент запускает событие `select`, где у объекта `event` есть свойство `category`. Любой компонент, который вложен в `<CategoryChooser>`, может прослушивать такие события:
<!-- When the user clicks a button, the component will fire a `select` event, where the `event` object has a `category` property. Any component that nests `<CategoryChooser>` can listen for events like so: -->

```html
<!--{ title: 'Component events' }-->
<CategoryChooser on:select="playTwentyQuestions(event.category)"/>

<script>
	import CategoryChooser from './CategoryChooser.html';

	export default {
		components: {
			CategoryChooser
		},

		methods: {
			playTwentyQuestions(category) {
				alert(`ok! you chose ${category}`);
			}
		}
	};
</script>
```

```html
<!--{ filename: 'CategoryChooser.html', hidden: true }-->
<p>Select a category:</p>

{#each categories as category}
	<button on:click="fire('select', { category })">select {category}</button>
{/each}

<script>
	export default {
		data() {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		}
	};
</script>
```

Так же, как `this` в обработчике событий элемента ссылается на сам элемент, так в обработчике события компонента `this` относится к компоненту, запускающему событие.
<!-- Just as `this` in an element's event handler refers to the element itself, in a component event handler `this` refers to the component firing the event. -->

Есть краткая запись для прослушивания и повторного запуска события без изменений.
<!-- There is also a shorthand for listening for and re-firing an event unchanged. -->

```html
<!-- { repl: false } -->
<!-- these are equivalent -->
<Widget on:foo="fire('foo', event)"/>
<Widget on:foo/>
```

Поскольку события компонента не распространяются как события DOM, это можно использовать для передачи событий через промежуточные компоненты. Эта сокращенная техника также применяется к событиям элемента (`on:click` эквивалентно: `on:click="fire('click', event)"`).
<!-- Since component events do not propagate as DOM events do, this can be used to pass events through intermediate components. This shorthand technique also applies to element events (`on:click` is equivalent to `on:click="fire('click', event)"`). -->


### Ссылки на узлы или компоненты (Refs)

Refs — удобный способ хранения ссылок на определенные DOM-узлы или компоненты. Объявите ref с помощью `ref:[name]` и получите доступ к нему внутри методов вашего компонента с помощью `this.refs.[name]`:
<!-- Refs are a convenient way to store a reference to particular DOM nodes or components. Declare a ref with `ref:[name]`, and access it inside your component's methods with `this.refs.[name]`: -->

```html
<!-- { title: 'Refs' } -->
<canvas ref:canvas width=200 height=200></canvas>

<script>
	import createRenderer from './createRenderer.js';

	export default {
		oncreate() {
			const canvas = this.refs.canvas;
			const ctx = canvas.getContext('2d');

			const renderer = createRenderer(canvas, ctx);
			this.on('destroy', renderer.stop);
		}
	}
</script>
```

```js
/* { filename: 'createRenderer.js', hidden: true } */
export default function createRenderer(canvas, ctx) {
	let running = true;
	loop();

	return {
		stop: () => {
			running = false;
		}
	};

	function loop() {
		if (!running) return;
		requestAnimationFrame(loop);

		const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

		for (let p = 0; p < imageData.data.length; p += 4) {
			const i = p / 4;
			const x = i % canvas.width;
			const y = i / canvas.height >>> 0;

			const t = window.performance.now();

			const r = 64 + (128 * x / canvas.width) + (64 * Math.sin(t / 1000));
			const g = 64 + (128 * y / canvas.height) + (64 * Math.cos(t / 1000));
			const b = 128;

			imageData.data[p + 0] = r;
			imageData.data[p + 1] = g;
			imageData.data[p + 2] = b;
			imageData.data[p + 3] = 255;
		}

		ctx.putImageData(imageData, 0, 0);
	}
}
```

> Поскольку только один элемент или компонент может занимать заданный `ref`, не используйте их в блоках `{#each ...}`. Однако, их можно использовать в блоках `{#if ...}`.
<!-- Since only one element or component can occupy a given `ref`, don't use them in `{#each ...}` blocks. It's fine to use them in `{#if ...}` blocks however. -->

Обратите внимание, что вы можете использовать refs в блоках `<style>` — см. [Специальные селекторы](guide#special-selectors).
<!-- Note that you can use refs in your `<style>` blocks — see [Special selectors](guide#special-selectors). -->


### Переходы (Transitions)

Переходы позволяют элементам плавно появляться и исчезать.
<!-- Transitions allow elements to enter and leave the DOM gracefully, rather than suddenly appearing and disappearing. -->

```html
<!-- { title: 'Transitions' } -->
<input type=checkbox bind:checked=visible> visible

{#if visible}
	<p transition:fade>fades in and out</p>
{/if}

<script>
	import { fade } from 'svelte-transitions';

	export default {
		transitions: { fade }
	};
</script>
```

Переходы могут иметь параметры — обычно это `delay`, `duration` и другие, в зависимости от рассматриваемого перехода. Например, вот переход `fly` из пакета [svelte-transitions](https://github.com/sveltejs/svelte-transitions):
<!-- Transitions can have parameters — typically `delay` and `duration`, but often others, depending on the transition in question. For example, here's the `fly` transition from the [svelte-transitions](https://github.com/sveltejs/svelte-transitions) package: -->

```html
<!-- { title: 'Transition with parameters' } -->
<input type=checkbox bind:checked=visible> visible

{#if visible}
	<p transition:fly="{y: 200, duration: 1000}">flies 200 pixels up, slowly</p>
{/if}

<script>
	import { fly } from 'svelte-transitions';

	export default {
		transitions: { fly }
	};
</script>
```

Элемент может иметь отдельные переходы `in` и `out`:
<!-- An element can have separate `in` and `out` transitions: -->

```html
<!-- { title: 'Transition in/out' } -->
<input type=checkbox bind:checked=visible> visible

{#if visible}
	<p in:fly="{y: 50}" out:fade>flies up, fades out</p>
{/if}

<script>
	import { fade, fly } from 'svelte-transitions';

	export default {
		transitions: { fade, fly }
	};
</script>
```

Переходы — это простые функции, которые берут «узел» и любые предоставленные параметры и возвращают объект со следующими свойствами:
<!-- Transitions are simple functions that take a `node` and any provided `parameters` and return an object with the following properties: -->

* `duration` — время перехода в миллисекундах
<!-- how long the transition takes in milliseconds -->
* `delay` — задержка перед началом в миллисекундах
<!-- milliseconds before the transition starts -->
* `easing` — an [easing function](https://github.com/rollup/eases-jsnext)
<!-- an [easing function](https://github.com/rollup/eases-jsnext) -->
* `css` — функция, которая принимает аргумент `t` равный числу между 0 и 1 и возвращает стили, которые должны применяться в этот момент
<!-- a function that accepts an argument `t` between 0 and 1 and returns the styles that should be applied at that moment -->
* `tick` — функция, которая будет вызываться в каждом кадре, с тем же аргументом `t`, пока выполняется переход
<!-- a function that will be called on every frame, with the same `t` argument, while the transition is in progress -->

Из них обязательный — `duration`, а так же либо `css`, либо `tick`. Остальные — необязательные. Вот как реализуется переход `fade`:
<!-- Of these, `duration` is required, as is *either* `css` or `tick`. The rest are optional. Here's how the `fade` transition is implemented, for example: -->

```html
<!-- { title: 'Fade transition' } -->
<input type=checkbox bind:checked=visible> visible

{#if visible}
	<p transition:fade>fades in and out</p>
{/if}

<script>
	export default {
		transitions: {
			fade(node, { delay = 0, duration = 400 }) {
				const o = +getComputedStyle(node).opacity;

				return {
					delay,
					duration,
					css: t => `opacity: ${t * o}`
				};
			}
		}
	};
</script>
```

> Если используется параметр `css`, Svelte создаст CSS-анимацию, которая эффективно работает с основным потоком. Поэтому, если вы можете добиться результата, используя `css`, а не `tick` — сделайте это.
<!-- If the `css` option is used, Svelte will create a CSS animation that runs efficiently off the main thread. Therefore if you can achieve an effect using `css` rather than `tick`, you should. -->

### Привязки (Bindings)

Как мы видели, данные могут быть переданы элементам и компонентам с атрибутами и [свойствами](guide#props). Иногда вам нужно вернуть данные — для этого мы используем привязки.
<!-- As we've seen, data can be passed down to elements and components with attributes and [props](guide#props). Occasionally, you need to get data back up; for that we use bindings. -->


#### Связывание компонентов (Component bindings)

Связи компонентов сохраняют значения при синхронизации между родителем и дочерним элементом:
<!-- Component bindings keep values in sync between a parent and a child: -->

```html
<!-- { repl: false } -->
<Widget bind:childValue=parentValue/>
```

Всякий раз, когда `childValue` изменяется в дочернем компоненте, `parentValue` будет обновляться в родительском компоненте и наоборот.
<!-- Whenever `childValue` changes in the child component, `parentValue` will be updated in the parent component and vice versa. -->

Если имена совпадают, вы можете использовать краткую запись:
<!-- If the names are the same, you can shorten the declaration: -->

```html
<!-- { repl: false } -->
<Widget bind:value/>
```

> Используйте связывание компонентов разумно. They can save you a lot of boilerplate, но вам будет сложнее следить за потоком данных в вашем приложении, если их будет слишком много.
<!-- Use component bindings judiciously. They can save you a lot of boilerplate, but will make it harder to reason about data flow within your application if you overuse them. -->

#### Связывание элементов (Element bindings)

Связывание элементов позволяет легко реагировать на взаимодействия пользователей:
<!-- Element bindings make it easy to respond to user interactions: -->

```html
<!-- { title: 'Element bindings' } -->
<h1>Hello {name}!</h1>
<input bind:value=name>
```

```json
/* { hidden: true } */
{
	name: 'world'
}
```

Некоторые привязки *односторонние*, что означает, что значения доступны только для чтения. Большинство из них *двусторонние* — изменение данных будет обновлять DOM. Доступны следующие привязки:
<!-- Some bindings are *one-way*, meaning that the values are read-only. Most are *two-way* — changing the data programmatically will update the DOM. The following bindings are available: -->

| Имя                                                            | Относится к                                  | Тип                        |
|----------------------------------------------------------------|----------------------------------------------|----------------------------|
| `value`                                                        | `<input>` `<textarea>` `<select>`            | <span>двусторонняя</span>  |
| `checked`                                                      | `<input type=checkbox>`                      | <span>двусторонняя</span>  |
| `group` (see note)                                             | `<input type=checkbox>` `<input type=radio>` | <span>двусторонняя</span>  |
| `currentTime` `paused` `played` `volume`                       | `<audio>` `<video>`                          | <span>двусторонняя</span>  |
| `buffered` `duration` `seekable`                               | `<audio>` `<video>`                          | <span>односторонняя</span> |
| `offsetWidth` `offsetHeight` `clientWidth` `clientHeight`      | Все блочные элементы                         | <span>односторонняя</span> |
| `scrollX` `scrollY`                                            | `<svelte:window>`                            | <span>двусторонняя</span>  |
| `online` `innerWidth` `innerHeight` `outerWidth` `outerHeight` | `<svelte:window>`                            | <span>односторонняя</span> |

> Групповое связывание ('group') позволяет вам фиксировать текущее значение [набора radio-инпутов](repl?demo=binding-input-radio) или всех выбранных значений [checkbox-инпутов](repl?demo=binding-input-checkbox-group).
<!-- 'group' bindings allow you to capture the current value of a [set of radio inputs](repl?demo=binding-input-radio), or all the selected values of a [set of checkbox inputs](repl?demo=binding-input-checkbox-group). -->

Вот полный пример использования двухсторонних привязок с формой:
<!-- Here is a complete example of using two way bindings with a form: -->

```html
<!-- { title: 'Form bindings' } -->
<form on:submit="handleSubmit(event)">
	<input bind:value=name type=text>
	<button type=submit>Say hello</button>
</form>

<script>
	export default {
		methods: {
			handleSubmit(event) {
				// prevent the page from reloading
				event.preventDefault();

				const { name } = this.get();
				alert(`Hello ${name}!`);
			}
		}
	};
</script>
```

```json
/* { hidden: true } */
{
	name: "world"
}
```

### Действия (Actions)

Действия позволяют вам оснащать элементы с дополнительной функциональностью. Действия — это функции, которые могут возвращать объект с помощью методов жизненного цикла: `update` и `destroy`. Действие вызывается когда его элемент добавляется в DOM.
<!-- Actions let you decorate elements with additional functionality. Actions are functions which may return an object with lifecycle methods, `update` and `destroy`. The action will be called when its element is added to the DOM. -->

Используйте действия для таких вещей, как:
* подсказки
* «ленивой» подгрузки картинок при скролле, например `<img use:lazyload data-src='giant-photo.jpg'/>`
* захвата ссылок для роутера (capturing link clicks for your client router)
* перетаскивание (drag and drop)
<!-- Use actions for things like:
* tooltips
* lazy loading images as the page is scrolled, e.g. `<img use:lazyload data-src='giant-photo.jpg'/>`
* capturing link clicks for your client router
* adding drag and drop -->

```html
<!-- { title: 'Actions' } -->
<button on:click="toggleLanguage()" use:tooltip="translations[language].tooltip">
	{language}
</button>

<script>
	export default {
		actions: {
			tooltip(node, text) {
				const tooltip = document.createElement('div');
				tooltip.textContent = text;

				Object.assign(tooltip.style, {
					position: 'absolute',
					background: 'black',
					color: 'white',
					padding: '0.5em 1em',
					fontSize: '12px',
					pointerEvents: 'none',
					transform: 'translate(5px, -50%)',
					borderRadius: '2px',
					transition: 'opacity 0.4s'
				});

				function position() {
					const { top, right, bottom } = node.getBoundingClientRect();
					tooltip.style.top = `${(top + bottom) / 2}px`;
					tooltip.style.left = `${right}px`;
				}

				function append() {
					document.body.appendChild(tooltip);
					tooltip.style.opacity = 0;
					setTimeout(() => tooltip.style.opacity = 1);
					position();
				}

				function remove() {
					tooltip.remove();
				}

				node.addEventListener('mouseenter', append);
				node.addEventListener('mouseleave', remove);

				return {
					update(text) {
						tooltip.textContent = text;
						position();
					},

					destroy() {
						tooltip.remove();
						node.removeEventListener('mouseenter', append);
						node.removeEventListener('mouseleave', remove);
					}
				}
			}
		},

		methods: {
			toggleLanguage() {
				const { language } = this.get();

				this.set({
					language: language === 'english' ? 'latin' : 'english'
				});
			}
		}
	};
</script>
```

```json
/* { hidden: true } */
{
	language: "english",
	translations: {
		english: {
			tooltip: "Switch Languages",
		},
		latin: {
			tooltip: "Itchsway Anguageslay",
		},
	}
}
```
