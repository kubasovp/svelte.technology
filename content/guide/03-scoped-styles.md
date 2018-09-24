---
title: Изолированные стили (Scoped styles)
---

Одним из ключевых принципов Svelte является то, что компоненты должны быть автономными и многоразовыми в разных контекстах. Из-за этого у него есть механизм для *изоляциии* вашего CSS, так что вы случайно не переопределяете другие селекторы на странице.
<!-- One of Svelte's key tenets is that components should be self-contained and reusable in different contexts. Because of that, it has a mechanism for *scoping* your CSS, so that you don't accidentally clobber other selectors on the page. -->

### Добавление стилей (Adding styles)

Ваш шаблон компонента может иметь тег `<style>`, например:
<!-- Your component template can have a `<style>` tag, like so: -->

```html
<!--{ title: 'Scoped styles' }-->
<div class="foo">
	Big red Comic Sans
</div>

<style>
	.foo {
		color: red;
		font-size: 2em;
		font-family: 'Comic Sans MS';
	}
</style>
```


### Как это работает (How it works)

Откройте приведенный выше пример в REPL и проверьте элемент, чтобы увидеть, что произошло: Svelte добавил класс `svelte-[uniqueid]` к элементу и соответствующим образом изменил селектор CSS. Поскольку ни один другой элемент на странице не может использовать этот селектор, все остальное на странице с `class="foo"` не будет затронуто нашими стилями.
<!-- Open the example above in the REPL and inspect the element to see what has happened – Svelte has added a `svelte-[uniqueid]` class to the element, and transformed the CSS selector accordingly. Since no other element on the page can share that selector, anything else on the page with `class="foo"` will be unaffected by our styles. -->

Это намного проще, чем достижение такого же эффекта с помощью [Shadow DOM](http://caniuse.com/#search=shadow%20dom) и работает везде без полифилов.
<!-- This is vastly simpler than achieving the same effect via [Shadow DOM](http://caniuse.com/#search=shadow%20dom) and works everywhere without polyfills. -->

> Svelte добавит тег `<style>` на страницу, содержащую ваши облаченные стили. Динамическое добавление стилей может быть невозможно, если на вашем сайте есть [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP). Если это так, вы можете использовать облачные стили с помощью [серверного рендеринга вашего CSS](guide#rendering-css) и использовать опцию компилятора `css: false` (или `--no-css` with the CLI).
<!-- Svelte will add a `<style>` tag to the page containing your scoped styles. Dynamically adding styles may be impossible if your site has a [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP). If that`s the case, you can use scoped styles by [server-rendering your CSS](guide#rendering-css) and using the `css: false` compiler option (or `--no-css` with the CLI). -->


### Каскадные правила (Cascading rules)

Стили будут применяться *только* к текущему компоненту, если вы не включите каскад с помощью модификатора `:global(...)`:
<!-- Styles will *only* apply to the current component, unless you opt in to cascading with the `:global(...)` modifier: -->

<!-- TODO `cascade: false` in the REPL -->

```html
<!-- { repl: false } -->
<div>
	<Widget/>
</div>

<style>
	p {
		/* this block will be disregarded, since
		   there are no <p> elements here */
		color: red;
	}

	div :global(p) {
		/* this block will be applied to any <p> elements
		   inside the <div>, i.e. in <Widget> */
		font-weight: bold;
	}
</style>

<script>
	import Widget from './Widget.html';

	export default {
		components: { Widget }
	};
</script>
```

> Изолированные стили *не* динамические - они разделяются между всеми экземплярами компонента. Другими словами, вы не можете использовать `{tags}` внутри своего CSS.
<!-- Scoped styles are *not* dynamic – they are shared between all instances of a component. In other words you can't use `{tags}` inside your CSS. -->


### Удаление неиспользуемых стилей (Unused style removal)

Svelte будет определять и удалять любые стили, если сможет гарантировать, что они не используются в вашем приложении. Он также выдаст предупреждение, чтобы вы могли удалить их из источника.
<!-- Svelte will identify and remove any styles that it can guarantee are not being used in your app. It will also emit a warning so that you can remove them from the source. -->


### Специальные селекторы (Special selectors)

Если у вас есть [ref](guide#refs) для элемента, вы можете использовать его как селектор в CSS. Селектор ``ref:*` имеет ту же специфику, что и селектор классов или атрибутов.
<!-- If you have a [ref](guide#refs) on an element, you can use it as a CSS selector. The `ref:*` selector has the same specificity as a class or attribute selector. -->


```html
<!--{ title: 'Styling with refs' }-->
<div ref:foo>
	yeah!
</div>

<style>
	ref:foo {
		color: magenta;
		font-size: 5em;
	}
</style>
```
