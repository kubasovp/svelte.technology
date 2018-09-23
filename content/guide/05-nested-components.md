---
title: Вложенные компоненты (Nested components)
---

Помимо вложенных элементов (и блоков `if` и `each`) компоненты Svelte могут содержать *другие* компоненты Svelte.
<!-- As well as containing elements (and `if` blocks and `each` blocks), Svelte components can contain *other* Svelte components. -->

```html
<!-- { title: 'Nested components' } -->
<div class='widget-container'>
	<Widget/>
</div>

<script>
	import Widget from './Widget.html';

	export default {
		components: {
			Widget
		}
	};
</script>
```

```html
<!--{ filename: 'Widget.html' }-->
<p>I am a nested component</p>
```

That's similar to doing this...

```js
import Widget from './Widget.html';

const widget = new Widget({
	target: document.querySelector('.widget-container')
});
```

...за исключением того, что Svelte позаботится об уничтожении дочернего компонента при уничтожении родителя и поддерживает синхронизацию двух компонентов со *свойствами*.
<!-- ...except that Svelte takes care of destroying the child component when the parent is destroyed, and keeps the two components in sync with *props*. -->

> Имена компонентов должны быть набраны ЗАГЛАВНЫМИ буквами в соответствии с широко используемым условным обозначением конструктора JavaScript. Это также простой способ отличить компоненты от элементов в вашем шаблоне.
<!-- Component names must be capitalised, following the widely-used JavaScript convention of capitalising constructor names. It's also an easy way to distinguish components from elements in your template. -->


### Props

Props - сокращение для 'properties' - средство, с помощью которого вы передаете данные от родителя к дочернему компоненту - иными словами, они похожи на атрибуты элемента:
<!-- Props, short for 'properties', are the means by which you pass data down from a parent to a child component — in other words, they're just like attributes on an element: -->

```html
<!--{ title: 'Props' }-->
<div class='widget-container'>
	<Widget foo bar="static" baz={dynamic}/>
</div>

<script>
	import Widget from './Widget.html';

	export default {
		components: {
			Widget
		}
	};
</script>
```

```html
<!--{ filename: 'Widget.html' }-->
<p>foo: {foo}</p>
<p>bar: {bar}</p>
<p>baz: {baz}</p>
```

```json
/* { hidden: true } */
{
	dynamic: 'try changing this text'
}
```

Как и атрибуты элементов, значения prop могут содержать любое допустимое выражение JavaScript.
<!-- As with element attributes, prop values can contain any valid JavaScript expression. -->

Часто имя свойства бывает таким же, как и значение - в этом случае мы можем использовать сокращенное название:
<!-- Often, the name of the property will be the same as the value, in which case we can use a shorthand: -->

```html
<!-- { repl: false } -->
<!-- these are equivalent -->
<Widget foo={foo}/>
<Widget {foo}/>
```

> Обратите внимание, что props *односторонние* - для получения данных из дочернего компонента в родительский компонент, используйте [bindings](guide#bindings).
<!-- Note that props are *one-way* — to get data from a child component into a parent component, use [bindings](guide#bindings). -->


### Испоьлзование `<slot>` (Composing with `<slot>`)

Компонент может содержать элемент `<slot></slot>`, который позволяет родительскому компоненту внедрять контент:
<!-- A component can contain a `<slot></slot>` element, which allows the parent component to inject content: -->

```html
<!-- { title: 'Using <slot>' } -->
<Box>
	<h2>Hello!</h2>
	<p>This is a box. It can contain anything.</p>
</Box>

<script>
	import Box from './Box.html';

	export default {
		components: { Box }
	};
</script>
```

```html
<!--{ filename: 'Box.html' }-->
<div class="box">
	<slot><!-- content is injected here --></slot>
</div>

<style>
	.box {
		border: 2px solid black;
		padding: 0.5em;
	}
</style>
```

Элемент `<slot>` может содержать 'резервный контент', который будет использоваться, если в компоненте нет дочерних элементов:
<!-- The `<slot>` element can contain 'fallback content', which will be used if no children are provided for the component: -->

```html
<!-- { title: 'Default slot content' } -->
<Box></Box>

<script>
	import Box from './Box.html';

	export default {
		components: { Box }
	};
</script>
```

```html
<!--{ filename: 'Box.html' }-->
<div class="box">
	<slot>
		<p class="fallback">the box is empty!</p>
	</slot>
</div>

<style>
	.box {
		border: 2px solid black;
		padding: 0.5em;
	}

	.fallback {
		color: #999;
	}
</style>
```

Вы также можете создавать именованные - *named* - слоты. Все элементы с соответствующим именем `slot` будут заполнять эти слоты:
You can also have *named* slots. Any elements with a corresponding `slot` attribute will fill these slots:

```html
<!-- { title: 'Named slots' } -->
<ContactCard>
	<span slot="name">P. Sherman</span>
	<span slot="address">42 Wallaby Way, Sydney</span>
</ContactCard>

<script>
	import ContactCard from './ContactCard.html';

	export default {
		components: { ContactCard }
	};
</script>
```

```html
<!--{ filename: 'ContactCard.html' }-->
<div class="contact-card">
	<h2><slot name="name"></slot></h2>
	<slot name="address">Unknown address</slot>
	<br>
	<slot name="email">Unknown email</slot>
</div>

<style>
	.contact-card {
		border: 2px solid black;
		padding: 0.5em;
	}
</style>
```


### Сокращенный импорт (Shorthand imports)

В качестве альтернативы декларации `import`...
<!-- As an alternative to using an `import` declaration... -->

```html
<!-- { repl: false } -->
<script>
	import Widget from './Widget.html';

	export default {
		components: { Widget }
	};
</script>
```

...вы можете написать это:
<!-- ...you can write this: -->

```html
<!-- { repl: false } -->
<script>
	export default {
		components: {
			Widget: './Widget.html'
		}
	};
</script>
```