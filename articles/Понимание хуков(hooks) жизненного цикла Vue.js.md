# Понимание хуков(hooks) жизненного цикла Vue.js

Крючки жизненного цикла являются важной частью любого серьезного компонента. Вам часто нужно знать, когда ваш компонент создается, добавляется в DOM, обновляется или уничтожается. Хуки жизненного цикла - это окно в то, как библиотека, которую вы используете, работает за кулисами и, как таковая, вызывает чувство трепета или беспокойства у новичков.

К счастью, это довольно легко понять, как видно на этой диаграмме. (Предоставлено [https://vuejs.org/](https://vuejs.org/).)

[![Vue.js Component Lifecycle Diagram](/images/b6749746ba4b05899f1e721a8084e572.png)](https://vuejs.org/v2/guide/instance.html)

## Создание (инициализация)

_Creation hooks_ - самые первые ловушки, которые запускаются в вашем компоненте. Они позволяют выполнять действия еще до того, как ваш компонент был добавлен в DOM. В отличие от любых других хуков, хуки создания также запускаются во время рендеринга на стороне сервера.

Используйте ловушки создания, если вам нужно настроить компоненты в компоненте как во время рендеринга клиента, так и рендеринга сервера. У вас не будет доступа к DOM или целевому элементу монтирования (_this.$el_) внутри хуков создания.

### beforeCreate

Хук _beforeCreate_ запускается при самой инициализации вашего компонента. Данные не были реактивными, а событие еще не настроено.

Пример:

ExampleComponent.vue

```javascript
export default {
  beforeCreate() {
    console.log('Nothing gets called before me!')
  }
}
```

### created

В ловушке _created_ вы сможете получить доступ к реактивным _data_ и _events_ активны. Шаблоны и Virtual DOM еще не были смонтированы или отображены.

Пример:

ExampleComponent.vue

```javascript
export default {
  data() {
    return {
      property: 'Blank'
    }
  },

  computed: {
    propertyComputed() {
      console.log('I change when this.property changes.')
      return this.property
    }
  },

  created() {
    this.property = 'Example property update.'
    console.log('propertyComputed will update, as this.property is now reactive.')
  }
}
```

## Монтаж (вставка DOM)

«Крепежные крючки»(mounting hooks) часто являются наиболее часто используемыми крючками, в лучшую или в худшую сторону. Они позволяют вам получить доступ к вашему компоненту непосредственно до и после первого рендера. Однако они не работают во время рендеринга на стороне сервера.

Используйте if: Вам нужно получить доступ или изменить DOM вашего компонента непосредственно перед или после первоначального рендеринга.

**не** использовать, если: Вам нужно получить некоторые данные для вашего компонента при инициализации. Вместо этого используйте _created_ (или _created_ + _activation_ для _keep-alive_ компонентов) для этого, особенно если вам нужны эти данные во время рендеринга на стороне сервера.

### beforeMount

Хук _beforeMount_ запускается непосредственно до того, как произойдет начальный рендеринг, и после компиляции шаблона или функций рендеринга. Скорее всего, вам никогда не понадобится этот крючок. Помните, это не вызывается при выполнении рендеринга на стороне сервера.

пример:

ExampleComponent.vue

```javascript
export default {
  beforeMount() {
    console.log(`this.$el doesn't exist yet, but it will soon!`)
  }
}
```

### mounted

In the_mounted_hook, you will have full access to the reactive component, templates, and rendered DOM (via. _this.$el_). Mounted is the most-often used lifecycle hook. The most frequently used patterns are fetching data for your component (use _created_ for this instead,) and modifying the DOM, often to integrate non-_Vue_ libraries.

Example:

ExampleComponent.vue

```
<template>
  <p>I'm text inside the component.</p>
</template>

<script>
export default {
  mounted() {
    console.log(this.$el.textContent) // I'm text inside the component.
  }
}
</script>

```

## [](https://alligator.io/vuejs/component-lifecycle/#updating-diff--re-render)Updating (Diff & Re-render)

_Updating hooks_are called whenever a reactive property used by your component changes, or something else causes it to re-render. They allow you to hook into the_watch-compute-render_ cycle for your component.

Use if: You need to know when your component re-renders, perhaps for debugging or profiling.

Do **not** use if: You need to know when a reactive property on your component changes. Use [computed properties](https://alligator.io/vuejs/computed-properties/) or [watchers](https://vuejs.org/v2/api/#watch) for that instead.

### beforeUpdate

The _beforeUpdate_ hook runs after data changes on your component and the update cycle begins, right before the DOM is patched and re-rendered. It allows you to get the new state of any reactive data on your component before it actually gets rendered.

Example:

ExampleComponent.vue

```
<script>
export default {
  data() {
    return {
      counter: 0
    }
  },

  beforeUpdate() {
    console.log(this.counter) // Logs the counter value every second, before the DOM updates.
  },

  created() {
    setInterval(() => {
      this.counter++
    }, 1000)
  }
}
</script>

```

### updated

The_updated_hook runs after data changes on your component and the DOM re-renders. If you need to access the DOM after a property change, here is probably the safest place to do it.

Example:

ExampleComponent.vue

```
<template>
  <p ref="dom-element">{{counter}}</p>
</template>
<script>
export default {
  data() {
    return {
      counter: 0
    }
  },

  updated() {
    // Fired every second, should always be true
    console.log(+this.$refs['dom-element'].textContent === this.counter)
  },

  created() {
    setInterval(() => {
      this.counter++
    }, 1000)
  }
}
</script>

```

## [](https://alligator.io/vuejs/component-lifecycle/#destruction-teardown)Destruction (Teardown)

_Destruction hooks_ allow you to perform actions when your component is destroyed, such as cleanup or analytics sending. They fire when your component is being torn down and removed from the DOM.

### beforeDestroy

_beforeDestroy_ is fired right before teardown. Your component will still be fully present and functional. If you need to cleanup events or reactive subscriptions,_beforeDestroy_ would probably be the time to do it.

Example:

ExampleComponent.vue

```
<script>
export default {
  data() {
    return {
      someLeakyProperty: 'I leak memory if not cleaned up!'
    }
  },

  beforeDestroy() {
    // Perform the teardown procedure for someLeakyProperty.
    // (In this case, effectively nothing)
    this.someLeakyProperty = null
    delete this.someLeakyProperty
  }
}
</script>

```

### destroyed

By the time you reach the_destroyed_hook, there’s pretty much nothing left on your component. Everything that was attached to it has been destroyed. You might use the _destroyed_ hook to do any last-minute cleanup or inform a remote server that the component was destroyed like a sneaky snitch.`<_<`

Example:

ExampleComponent.vue

```
<script>
import MyCreepyAnalyticsService from './somewhere-bad'

export default {
  destroyed() {
    console.log(this) // There's practically nothing here!
    MyCreepyAnalyticsService.informService('Component destroyed. All assets move in on target on my mark.')
  }
}
</script>

```

## Other Hooks

There are two other hooks,_activated_ and _deactivated_. These are for _keep-alive_ components, a topic that is outside the scope of this article. Suffice it to say that they allow you to detect when a component that is wrapped in a _<keep-alive></keep-alive>_ tag is toggled on or off. You might use them to fetch data for your component or handle state changes, effectively behaving as _created_ and _beforeDestroy_ without the need to do a full component rebuild.