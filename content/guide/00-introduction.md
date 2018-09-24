---
title: Прелюдия (Introduction)
---

### Что такое Svelte? (What is Svelte?)

Если вы когда-либо создавали приложение для JavaScript, скорее всего, вы сталкивались – или, по крайней мере, слышали о таких фреймворках, как React, Angular, Vue и Ractive. Как и Svelte, все эти инструменты имеют целью упростить создание интерактивных пользовательских интерфейсов.
<!-- If you've ever built a JavaScript application, the chances are you've encountered – or at least heard of – frameworks like React, Angular, Vue and Ractive. Like Svelte, these tools all share a goal of making it easy to build slick interactive user interfaces. -->

Но у Svelte есть ключевое отличие: вместо интерпретации кода приложения во *время выполнения*, ваше приложение преобразуется в идеальный JavaScript во *время сборки*. Это означает, что вы не оплачиваете стоимость выполнения абстракций фреймворка или не загружаете много ненужного кода при первоначальной загрузке приложения.
<!-- But Svelte has a crucial difference: rather than interpreting your application code at *run time*, your app is converted into ideal JavaScript at *build time*. That means you don't pay the performance cost of the framework's abstractions, or incur a penalty when your app first loads. -->

И, поскольку накладных расходов нет, вы можете легко внедрить Svelte в существующее приложение поэтапно или исрпользовать виджеты в виде автономных пакетов, которые работают в любом месте.
<!-- And because there's no overhead, you can easily adopt Svelte in an existing app incrementally, or ship widgets as standalone packages that work anywhere. -->

[Читайте вступительную запись в блоге](/blog/frameworks-without-the-framework), чтобы узнать больше о целях и философии Svelte.
<!-- [Read the introductory blog post](/blog/frameworks-without-the-framework) to learn more about Svelte's goals and philosophy. -->


### Компоненты в Svelte (Understanding Svelte components)

В Svelte приложение составлено из одного или нескольких *компонентов*. Компонент представляет собой многократно используемый автономный блок кода, в котором инкапсулированы разметка, стили и поведения, которые принадлежат друг другу.
<!-- In Svelte, an application is composed from one or more *components*. A component is a reusable self-contained block of code that encapsulates markup, styles and behaviours that belong together. -->

Подобно Ractive и Vue, Svelte продвигает концепцию *однофайловых компонентов*: компонент является одним `.html` файлом. Вот простой пример:
<!-- Like Ractive and Vue, Svelte promotes the concept of *single-file components*: a component is just an `.html` file. Here's a simple example: -->

```html
<!--{ title: 'Hello world!' }-->
<h1>Hello {name}!</h1>
```

```json
/* { hidden: true } */
{
	name: 'world'
}
```

> Где бы вы ни увидели ссылки на <strong style="font-weight: 700; font-size: 16px; font-family: Inconsolata, monospace; color: rgba(170,30,30, 0.8)">REPL</strong> – нажмите для интерактивного примера
<!-- Wherever you see <strong style="font-weight: 700; font-size: 16px; font-family: Inconsolata, monospace; color: rgba(170,30,30, 0.8)">REPL</strong> links, click through for an interactive example -->

Svelte превращает это в JavaScript-модуль, который вы можете импортировать в свое приложение:
<!-- Svelte turns this into a JavaScript module that you can import into your app: -->

```js
/* { filename: 'main.js' } */
import App from './App.html';

const app = new App({
	target: document.querySelector('main'),
	data: { name: 'world' }
});

// change the data associated with the template
app.set({ name: 'everybody' });

// detach the component and clean everything up
app.destroy();
```

Поздравляем, вы только что узнали половину API Svelte!
<!-- Congratulations, you've just learned about half of Svelte's API! -->


### Начало работы (Getting started)

Обычно в этой части гайда вам предлагают подключить фреймворк на вашу страницу с помощью тега <script>. Но поскольку Svelte работает во время сборки, тут немного инче.
<!-- Normally, this is the part where the instructions might tell you to add the framework to your page as a `<script>` tag. But because Svelte runs at build time, it works a little bit differently. -->

Лучший способ использовать Svelte – это интегрировать его в вашу систему сборки – есть плагины для Rollup, Browserify, Gulp и на очереди ещё ряд других. См. [Актуальный список](https://github.com/sveltejs/svelte/#svelte).
<!-- The best way to use Svelte is to integrate it into your build system – there are plugins for Rollup, Browserify, Gulp and others, with more on the way. See [here](https://github.com/sveltejs/svelte/#svelte) for an up-to-date list. -->

> Вам нужно будет установить [Node.js](https://nodejs.org/en/) и ознакомиться с командной строкой
<!-- You will need to have [Node.js](https://nodejs.org/en/) installed, and have some familiarity with the command line -->

#### Начало работы с REPL (Getting started using the REPL)

Перейдя в [REPL](/repl) и нажав кнопку *download* на любом из примеров, вы получите ZIP-файл, содержащий все, что вам нужно для запуска этого примера локально. Просто разархивируйте его, `cd` в каталог и запустите` npm install` и `npm run dev`. См. [эту заметку в блоге](/blog/the-easiest-way-to-get-started) для получения дополнительной информации.
<!-- Going to the [REPL](/repl) and pressing the *download* button on any of the examples will give you a .zip file containing everything you need to run that example locally. Just unzip it, `cd` to the directory, and run `npm install` and `npm run dev`. See [this blog post](/blog/the-easiest-way-to-get-started) for more information. -->

#### Начало работы с degit (Getting started using degit)

[degit](https://github.com/Rich-Harris/degit) это инструмент для создания проектов из шаблонов, хранящихся в git-репозиториях. Установите его глобально...
<!-- [degit](https://github.com/Rich-Harris/degit) is a tool for creating projects from templates stored in git repos. Install it globally... -->

```bash
npm install -g degit
```

... и тогда вы сможете использовать его для создания нового проекта:
<!-- ...then you can use it to spin up a new project: -->

```bash
degit sveltejs/template my-new-project
cd my-new-project
npm install
npm run dev
```

Вы можете использовать любой git-репозиторий, который вам нравится - это «официальные» шаблоны:
<!-- You can use any git repo you like — these are the 'official' templates: -->

* [sveltejs/template](https://github.com/sveltejs/template) — здесь то, что вы получите, загрузив пример из REPL
<!-- this is what you get by downloading from the REPL -->
* [sveltejs/template-webpack](https://github.com/sveltejs/template-webpack) — аналогично, но использует [webpack](https://webpack.js.org/) вместо [Rollup](https://rollupjs.org/guide/en)
<!-- similar, but uses [webpack](https://webpack.js.org/) instead of [Rollup](https://rollupjs.org/guide/en) -->

#### Начало работы с CLI (Getting started using the CLI)

Svelte также предоставляет интерфейс командной строки, но он не рекомендуется для использования в производстве. CLI будет компилировать ваши компоненты в автономные файлы JavaScript, но не будет автоматически перекомпилировать их при их изменении и не будет дедуплицировать код, разделяемый между вашими компонентами. Вместо этого используйте один из вышеперечисленных методов.
<!-- Svelte also provides a Command Line Interface, but it's not recommended for production use. The CLI will compile your components to standalone JavaScript files, but won't automatically recompile them when they change, and won't deduplicate code shared between your components. Use one of the above methods instead. -->

Если вы установили `svelte` глобально, вы можете использовать `svelte --help` для получения полного списка опций. Некоторые примеры операций:
<!-- If you've installed `svelte` globally, you can use `svelte --help` for a complete list of options. Some examples of the more common operations are: -->

```bash
# Generate a JavaScript module from MyComponent.html
svelte compile MyComponent.html > MyComponent.js
svelte compile -i MyComponent.html -o MyComponent.js

# Generate a UMD module from MyComponent.html, inferring its name from the filename ('MyComponent')
svelte compile -f umd MyComponent.html > MyComponent.js

# Generate a UMD module, specifying the name
svelte compile -f umd -n CustomName MyComponent.html > MyComponent.js

# Compile all .html files in a directory
svelte compile -i src/components -o build/components
```

> Вы можете использовать [npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b) для использования CLI без глобальной установки Svelte — используйте префикс `npx` для вашей команды: `npx svelte compile ...`
<!-- You can also use [npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b) to use the CLI without installing Svelte globally — just prefix your command with `npx`: `npx svelte compile ...` -->
