---
title: Разное (Miscellaneous)
---


### `<noscript>`

Если вы используете теги `<noscript>` в компоненте, Svelte будет отображать их только в режиме SSR. Компилятор DOM отключит их, поскольку вы не можете создать компонент без JavaScript, а `<noscript>` не имеет никакого эффекта, если доступен JavaScript.
<!-- If you use `<noscript>` tags in a component, Svelte will only render them in SSR mode. The DOM compiler will strip them out, since you can't create the component without JavaScript, and `<noscript>` has no effect if JavaScript is available. -->