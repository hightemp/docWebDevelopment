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

Используйте если: Вам нужно получить доступ или изменить DOM вашего компонента непосредственно перед или после первоначального рендеринга.

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

В ловушке _mounted_ у вас будет полный доступ к реактивному компоненту, шаблонам и отображаемому DOM (через. _this.$el_). Установленный - наиболее часто используемый крюк жизненного цикла. Наиболее часто используемые шаблоны - это выборка данных для вашего компонента (вместо этого используйте _created_) и изменение DOM, часто для интеграции не-_Vue_ библиотек.

Пример:

ExampleComponent.vue

```html
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

## Обновление (Diff & Re-render)

_Updating hooks_ вызываются всякий раз, когда изменяется реактивное свойство, используемое вашим компонентом, или что-то еще вызывает его повторную визуализацию. Они позволяют вам подключиться к циклу _watch-compute-render_ для вашего компонента.

Используйте если: вам нужно знать, когда ваш компонент будет перерисован, возможно, для отладки или профилирования.

**не** используйте, если: Вам необходимо знать, когда изменяется реактивное свойство вашего компонента. Используйте для этого [вычисленные свойства](https://alligator.io/vuejs/computed-properties/) или [watchers](https://vuejs.org/v2/api/#watch).

### beforeUpdate

Хук _beforeUpdate_ запускается после изменения данных в вашем компоненте и запускается цикл обновления, непосредственно перед исправлением и повторной визуализацией DOM. Это позволяет вам получить новое состояние любых реактивных данных в вашем компоненте до того, как они будут фактически отображены.

Пример:

ExampleComponent.vue

```javascript
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
```

### updated

Хук _updated_ запускается после изменения данных в вашем компоненте и повторного рендеринга DOM. Если вам нужно получить доступ к DOM после изменения свойства, это, вероятно, самое безопасное место для этого.

Пример:

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

## Уничтожение (Разрушение)

_Destruction hooks_ позволяет вам выполнять действия, когда ваш компонент уничтожен, такие как очистка или отправка аналитики. Они срабатывают, когда ваш компонент снят и удален из DOM.

### beforeDestroy

_beforeDestroy_ активирован прямо перед сносом. Ваш компонент по-прежнему будет полностью представлен и функционален. Если вам нужно очистить события или реактивные подписки, _beforeDestroy_, вероятно, самое время сделать это.

Пример:

ExampleComponent.vue

```javascript
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
```

### destroyed

К тому времени, когда вы достигнете _destroyed_ хука, в вашем компоненте уже почти ничего не останется. Все, что было прикреплено к нему, было уничтожено. Вы можете использовать ловушку _destroyed_, чтобы выполнить очистку в последнюю минуту или сообщить удаленному серверу, что компонент был уничтожен, как хитрая стукачка. <_<

Пример:

ExampleComponent.vue

```javascript
import MyCreepyAnalyticsService from './somewhere-bad'

export default {
  destroyed() {
    console.log(this) // There's practically nothing here!
    MyCreepyAnalyticsService.informService('Component destroyed. All assets move in on target on my mark.')
  }
}
```

## Другие хуки

Есть два других хука, _acactive_ и _deactivation_. Они предназначены для компонентов _keep-alive_, тема которых выходит за рамки данной статьи. Достаточно сказать, что они позволяют вам определять, когда компонент, включенный в тег _<keep-alive></ keep-alive>_, включен или выключен. Вы можете использовать их для извлечения данных для вашего компонента или обработки изменений состояния, фактически ведя себя как _created_ и _beforeDestroy_ без необходимости полной перестройки компонента.

**********
[Vue](/tags/Vue.md)
[hooks](/tags/hooks.md)
