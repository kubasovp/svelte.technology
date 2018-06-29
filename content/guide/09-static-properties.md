---
title: Статические свойства
---


### Установка (Setup)

В некоторых ситуациях вы можете добавить статические свойства в свой конструктор компонента. Для этого мы используем свойство `setup`:
<!-- In some situations, you might want to add static properties to your component constructor. For that, we use the `setup` property: -->

```html
<!-- { title: 'Component setup' } -->
<p>check the console!</p>

<script>
	export default {
		setup(MyComponent) {
			// someone importing this component will be able
			// to access any properties or methods defined here:
			//
			//   import MyComponent from './MyComponent.html';
			//   console.log(MyComponent.ANSWER); // 42
			MyComponent.ANSWER = 42;
		},

		oncreate() {
			console.log('the answer is', this.constructor.ANSWER);
			console.dir(this.constructor);
		}
	};
</script>
```

### Предварительная загрузка (preload)

Если описание вашего компонента включает функцию предварительной загрузки — `preload`, он будет привязан к конструктору компонента как статический метод. Это никак не меняет поведение вашего компонента — это соглашение, которое позволяет системам, таким как [Sapper](https://sapper.svelte.technology), откладывать рендеринг компонента до тех пор, пока его данные не будут готовы.
<!-- If your component definition includes a `preload` function, it will be attached to the component constructor as a static method. It doesn't change the behaviour of your component in any way — instead, it's a convention that allows systems like [Sapper](https://sapper.svelte.technology) to delay rendering of a component until its data is ready. -->

Дополнительную информацию см. в разделе [preloading](https://sapper.svelte.technology/guide#preloading) документации Sapper.
<!-- See the section on [preloading](https://sapper.svelte.technology/guide#preloading) in the Sapper docs for more information. -->
