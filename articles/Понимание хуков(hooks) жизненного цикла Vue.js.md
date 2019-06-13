# Понимание хуков(hooks) жизненного цикла Vue.js

Lifecycle hooks are an important part of any serious component. You often need to know when your component is created, added to the DOM, updated, or destroyed. Lifecycle hooks are a window into how the library you’re using works behind-the-scenes, and as such tend to invoke feelings of trepidation or unease for newcomers.

Thankfully, it’s fairly easy to understand, as seen in this diagram. (Courtesy of [https://vuejs.org/](https://vuejs.org/).)

[![Vue.js Component Lifecycle Diagram](/images/b6749746ba4b05899f1e721a8084e572.png)](https://vuejs.org/v2/guide/instance.html)

## [](https://alligator.io/vuejs/component-lifecycle/#creation-initialization)Creation (Initialization)

_Creation hooks_are the very first hooks that run in your component. They allow you to perform actions before your component has even been added to the DOM. Unlike any of the other hooks, creation hooks are also run during server-side rendering.

Use creation hooks if you need to set things up in your component both during client rendering and server rendering. You will not have access to the DOM or the target mounting element (_this.$el_) inside of creation hooks.

### beforeCreate

The_beforeCreate_hook runs at the very initialization of your component._data_has not been made reactive, and_events_have not been set up yet.

Example:

ExampleComponent.vue

```
<script>
export default {
  beforeCreate() {
    console.log('Nothing gets called before me!')
  }
}
</script>

```

### created

In the_created_hook, you will be able to access reactive_data_and_events_are active. Templates and Virtual DOM have not yet been mounted or rendered.

Example:

ExampleComponent.vue

```
<script>
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
</script>

```

## [](https://alligator.io/vuejs/component-lifecycle/#mounting-dom-insertion)Mounting (DOM Insertion)

_Mounting hooks_are often the most-used hooks, for better or worse. They allow you to access your component immediately before and after the first render. They do not, however, run during server-side rendering.

Use if: You need to access or modify the DOM of your component immediately before or after the initial render.

Do**not**use if: You need to fetch some data for your component on initialization. Use_created_(or_created_+_activated_for_keep-alive_components) for this instead, especially if you need that data during server-side rendering.

### beforeMount

The_beforeMount_hook runs right before the initial render happens and after the template or render functions have been compiled. Most likely you’ll never need to use this hook. Remember, it doesn’t get called when doing server-side rendering.

Example:

ExampleComponent.vue

```
<script>
export default {
  beforeMount() {
    console.log(`this.$el doesn't exist yet, but it will soon!`)
  }
}
</script>

```

### mounted

In the_mounted_hook, you will have full access to the reactive component, templates, and rendered DOM (via._this.$el_). Mounted is the most-often used lifecycle hook. The most frequently used patterns are fetching data for your component (use_created_for this instead,) and modifying the DOM, often to integrate non-_Vue_libraries.

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

_Updating hooks_are called whenever a reactive property used by your component changes, or something else causes it to re-render. They allow you to hook into the_watch-compute-render_cycle for your component.

Use if: You need to know when your component re-renders, perhaps for debugging or profiling.

Do**not**use if: You need to know when a reactive property on your component changes. Use[computed properties](https://alligator.io/vuejs/computed-properties/)or[watchers](https://vuejs.org/v2/api/#watch)for that instead.

### beforeUpdate

The_beforeUpdate_hook runs after data changes on your component and the update cycle begins, right before the DOM is patched and re-rendered. It allows you to get the new state of any reactive data on your component before it actually gets rendered.

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

_Destruction hooks_allow you to perform actions when your component is destroyed, such as cleanup or analytics sending. They fire when your component is being torn down and removed from the DOM.

### beforeDestroy

_beforeDestroy_is fired right before teardown. Your component will still be fully present and functional. If you need to cleanup events or reactive subscriptions,_beforeDestroy_would probably be the time to do it.

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

By the time you reach the_destroyed_hook, there’s pretty much nothing left on your component. Everything that was attached to it has been destroyed. You might use the_destroyed_hook to do any last-minute cleanup or inform a remote server that the component was destroyed like a sneaky snitch.`<_<`

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