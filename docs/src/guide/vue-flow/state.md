# State

Under the hood Vue Flow uses [Provide/Inject](https://v3.vuejs.org/guide/component-provide-inject)
to pass around it's state between components.
You can access the internal state through the [`useVueFlow`](/guide/composables#usevueflow/) composable.

[`useVueFlow`](/guide/composables#usevueflow/) can be used to either create a new state instance and inject it into
the current component tree or inject
an already existing store from the current context.
Internal state can be manipulated, for example by adding new elements to the state. The
state is reactive and changes will be reflected on the graph.

```vue
<script setup>
import { useVueFlow } from '@vue-flow/core'

const { getNodes, onPaneReady } = useVueFlow()

// event handler
onPaneReady((i) => i.fitView())

// watch the stored nodes
watch(getNodes, (nodes) => console.log('nodes changed', nodes))
</script>
```

## Accessing Internal State

Using the composition API also allows us to pass the state around outside the current component context, thus we have a
lot more flexibility when it comes
to reading, writing and updating the state.

Consider this example, where we want to create a Sidebar that allows us to select all nodes.

```vue
<!-- Container.vue -->
<template>
  <div>
    <Sidebar />
    <div class="wrapper">
      <VueFlow :nodes="nodes" :edges="edges" />
    </div>
  </div>
</template>
```

We could pass all necessary info as props to the Sidebar, which could become either tedious or result in prop drilling,
which we want to avoid.
In this example it wouldn't be a big issue but if our destination was 3 components deep, it would become hard to track
the flow of information.

Instead, we can initialize a Vue Flow store instance __before__ the Sidebar is initialized, thus the instance becomes
available as an injection in the component tree.

```vue{5-6}
<script>
// Container.vue
import { useVueFlow  } from '@vue-flow/core'

// initialize a store instance in this context, so it is available when calling inject(VueFlow)
useVueFlow()
</script>
```

Now we can easily access our current state instance from our Sidebar without passing them as props.

```vue
<script setup>
import { useVueFlow } from '@vue-flow/core'

const { nodesSelectionActive, addSelectedNodes, getNodes } = useVueFlow()

const selectAll = () => {
  addSelectedNodes(getNodes.value)
  nodesSelectionActive.value = true
}
</script>
<template>
  <aside>
    <div class="description">
      This is an example of how you can access the internal state outside of the Vue VueFlow component.
    </div>
    <div class="selectall">
      <button @click="selectAll">select all nodes</button>
    </div>
  </aside>
</template>
```

::: tip
If you have multiple store instances in the same context, make sure to give them a unique id in order to guarantee
access to the correct instance.
Otherwise `useVueFlow` will try to inject the first instance it can find in the current context, which would usually be
the last one that has been injected.
:::

## State Updates

State updates like removing elements or updating positions are applied by default.
If you want to strictly control state changes you can disable this behavior by setting the `applyDefault` option/prop
to `false`.

```vue
<template>
    <VueFlow :nodes="nodes" :edges="edges" :apply-default="false" />
</template>
```

State changes are emitted by the `onNodesChange` or `onEdgesChange` events, which will provide an array of changes that
have been triggered.
To take control of state changes you can implement your own state update handlers or use the state helper functions that
come with the library to mix it up.

::: info
Read more about this in the [controlled flow](/guide/controlled-flow) guide.
:::

## Access State in Options API

`useVueFlow` was designed to be used in the composition API, __but__ it is still possible to use it in the options API.
Though it is necessary to pass a unique id for your Vue Flow state instance, otherwise a look-up will fail and Vue Flow
will create a new state instance
when mounted.

```vue
<script>
import { VueFlow, useVueFlow } from '@vue-flow/core'

const { addEdges, onConnect } = useVueFlow({ id: 'options-api' })
export default defineComponent({
  components: { VueFlow },
  data() {
    return {
      nodes: [
        {
          id: '1',
          position: { x: 0, y: 0},
          data: { label: 'Node 1' }
        }
      ],
      edges: [],
    }
  },
  methods: {
    // regular event handler
    handleConnect: (params) => {
      addEdges([params])
    }
  },
  beforeMount() {
    // Register your event handler, can technically be called in any lifecycle phase
    // Skip this if you're using regular event handlers
    onConnect((params) => addEdges([params]))
  }
})
</script>

<template>
  <VueFlow id="options-api" :nodes="nodes" :edges="edges" @connect="handleConnect" />
</template>
```
