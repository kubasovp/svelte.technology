---
title: Пользовательские элементы (Custom elements)
---

[Пользовательские элементы](https://developer.mozilla.org/ru/docs/Web/Web_Components/Использование_пользовательских_элементов) — это стандарт создания элементов DOM, которые инкапсулируют стили и поведение, подобно компонентам Svelte. Они являются частью семейства спецификаций [веб-компоненты](https://developer.mozilla.org/ru/docs/Web/Web_Components).
<!-- [Custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements) are an emerging web standard for creating DOM elements that encapsulate styles and behaviours, much like Svelte components. They are part of the [web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) family of specifications. -->

> Большинство браузеров нуждаются в [polyfills](https://www.webcomponents.org/polyfills) для пользовательских элементов. См. [caniuse](https://caniuse.com/#feat=custom-elementsv1) для получения более подробной информации
<!-- Most browsers need [polyfills](https://www.webcomponents.org/polyfills) for custom elements. See [caniuse](https://caniuse.com/#feat=custom-elementsv1) for more details -->

Компоненты Svelte могут использоваться как пользовательские элементы, выполняя следующие действия:
<!-- Svelte components can be used as custom elements by doing the following: -->

1. Объявление имени `тега`. Значение должно содержать дефис (`hello-world` в приведенном ниже примере)
2. Указание `customElement: true` в конфигурации компилятора
<!-- 1. Declaring a `tag` name. The value must contain a hyphen (`hello-world` in the example below)
2. Specifying `customElement: true` in the compiler configuration -->

```html
<!-- { filename: 'HelloWorld.html', repl: false } -->
<h1>Hello {name}!</h1>

<script>
	export default {
		tag: 'hello-world'
	};
</script>
```

При импорте этого файла будет зарегистрирован глобальный пользовательский элемент `<hello-world>`, который принимает свойство `name`:
<!-- Importing this file will now register a globally-available `<hello-world>` custom element that accepts a `name` property: -->

```js
import './HelloWorld.html';
document.body.innerHTML = `<hello-world name="world"/>`;

const el = document.querySelector('hello-world');
el.name = 'everybody';
```

См. [svelte-custom-elements.surge.sh](http://svelte-custom-elements.surge.sh/) ([источник](https://github.com/sveltejs/template-custom-element)) для более подробного примера.
<!-- See [svelte-custom-elements.surge.sh](http://svelte-custom-elements.surge.sh/) ([source here](https://github.com/sveltejs/template-custom-element)) for a larger example. -->

Скомпилированные пользовательские элементы так же являются полнофункциональными компонентами Svelte и могут использоваться как таковые:
<!-- The compiled custom elements are still full-fledged Svelte components and can be used as such: -->

```js
el.get().name === el.name; // true
el.set({ name: 'folks' }); // equivalent to el.name = 'folks'
```

Одно существенное различие заключается в том, что стили *полностью инкапсулированы* — в то время как Svelte предотвращает утечку стилей компонентов из *out*, пользовательские элементы используют [shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Shadow_DOM), который также предотвращает утечку стилей из *in*.
<!-- One crucial difference is that styles are *fully encapsulated* — whereas Svelte will prevent component styles from leaking *out*, custom elements use [shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Shadow_DOM) which also prevents styles from leaking *in*. -->

### Использование `<slot>`

Пользовательские элементы могут использовать [slots](guide#composing-with-slot) для размещения дочерних элементов как обычные компоненты Svelte.
<!-- Custom elements can use [slots](guide#composing-with-slot) to place child elements, just like regular Svelte components. -->

### Firing events

Вы можете отправлять события внутри пользовательских элементов для передачи данных:
<!-- You can dispatch events inside custom elements to pass data out: -->

```js
// inside a component method
const event = new CustomEvent('message', {
	detail: 'Hello parent!',
	bubbles: true,
	cancelable: true,
	composed: true // makes the event jump shadow DOM boundary
});

this.dispatchEvent(event);
```

Другие части приложения могут прослушивать эти события с помощью `addEventListener`:
<!-- Other parts of the application can listen for these events with `addEventListener`: -->

```js
const el = document.querySelector('hello-world');
el.addEventListener('message', event => {
	alert(event.detail);
});
```

> Обратите внимание на атрибут `composed: true` настраиваемого события. Он позволяет пользовательскому событию DOM пересекать границу теневого DOM и входить в основное дерево DOM.
<!-- Note the `composed: true` attribute of the custom event. It enables the custom DOM event to cross the shadow DOM boundary and enter into main DOM tree. -->

### Видимость свойств (Observing properties)

Svelte определит из шаблона и значений `computed`, какие свойства имеет пользовательский элемент — например, `name` в нашем примере `<hello-world>`. Вы можете прописать список свойств вручную, чтобы указать, какие из них являются 'видимыми' для остальной части вашего приложения:
<!-- Svelte will determine, from the template and `computed` values, which properties the custom element has — for example, `name` in our `<hello-world>` example. You can specify this list of properties manually, for example to restrict which properties are 'visible' to the rest of your app: -->

```js
export default {
	tag: 'my-thing',
	props: ['foo', 'bar']
};
```

### Параметры компилятора (Compiler options)

Раньше мы видели использование `customElement: true`, чтобы указать компилятору Svelte создать пользовательский элемент используя `tag` и (опционально) `props`, объявленные внутри компонента.
<!-- Earlier, we saw the use of `customElement: true` to instruct the Svelte compiler to generate a custom element using the `tag` and (optional) `props` declared inside the component file. -->

Кроме того, вы можете передать `tag` и `props` прямо в компилятор:
<!-- Alternatively, you can pass `tag` and `props` direct to the compiler: -->

```js
const { js } = svelte.compile(source, {
	customElement: {
		tag: 'overridden-tag-name',
		props: ['yar', 'boo']
	}
});
```

Эти параметры будут переопределять собственные настройки компонента, если таковые имеются.
<!-- These options will override the component's own settings, if any. -->

### Транспиляция (Transpiling)

* Пользовательские элементы используют классы ES2015 (`MyThing extends HTMLElement`). Убедитесь, что вы не перенаправляете код пользовательского элемента в ES5 и используете минификатор, поддерживающий ES2015, например [uglify-es](https://www.npmjs.com/package/uglify-es).
<!-- Custom elements use ES2015 classes (`MyThing extends HTMLElement`). Make sure you don't transpile the custom element code to ES5, and use a ES2015-aware minifier such as [uglify-es](https://www.npmjs.com/package/uglify-es). -->

* Если вам нужна поддержка ES5, обязательно используйте плагин с поддержкой `Reflect.construct`, такой как [babel-plugin-transform-builtin-classes](https://github.com/WebReflection/babel-plugin-transform-builtin-classes) и полифил, например [custom-elements-es5-adapterjs](https://github.com/webcomponents/webcomponentsjs#custom-elements-es5-adapterjs).
<!-- If you do need ES5 support, make sure to use `Reflect.construct` aware transpiler plugin such as [babel-plugin-transform-builtin-classes](https://github.com/WebReflection/babel-plugin-transform-builtin-classes) and a polyfill like [custom-elements-es5-adapterjs](https://github.com/webcomponents/webcomponentsjs#custom-elements-es5-adapterjs). -->
