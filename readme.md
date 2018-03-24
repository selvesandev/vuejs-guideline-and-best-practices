# Vue js Style Guide and Best Practices

#### Guide for developing Vue.js applications.

> This is the style guide that is put together for the team to help organize ourselves as we move from a primarily server-side rendered, Cold-Fusion based view to dynamic SPAs using Vue.


## Section 1: The Basics.
Vue.js has put out an [official style guide](https://vuejs.org/v2/style-guide/) and we should consider this document an extension of that one.  While this shouldn't be an exhaustive list, here are some key takeaways: 

* Component data() must be a function, so that components can be reused with new instances of components. 
* Prop definitions should be very detailed, including, if possible, 'type', 'required', and 'validator' information. 

Bad:
```javascript 
Vue.component({
  props: ['status']
})
```

Good: 
``` 
Vue.component({
  props: {
    status: {
      type: String,
      required: true,
      validator: (value) => ['syncing', 'synced', 'error'].includes(value); 
    }
  }
})
```

* Always use key with v-for.
* No more than one component per file.
* Use PascalCase for naming component files. (i.e. "MyComponent.vue" not "myComponent.vue")
* Base components (that is, a "dumb" component that has no interactivity and just displays data) should begin with the prefix "Base". (As in : "BaseButton.vue", "BaseTable.vue")
* Components that are created and used only once should be prefixed with "The", as in "TheHeading.vue, "TheSidebar.vue"
* Child components tightly coupled with their parent should include the parent component name as a prefix. For example, "TodoList.vue" has children "TodoListItem.vue" & "TodoListAddButton.vue"
* Directives and template expressions should only contain the most simple of javascript commands.

Bad: 
```html
<template>
  <div>
    <app-some-component @click="val => val.reduce((pv, cv) => {
      pv[cv.id] = cv;
      return pv; 
    }, {})">
    </app-some-component>
      {{ fullName.split(' ').map(function (word) {
        return word[0].toUpperCase() + word.slice(1)
      }).join(' ')
    }}
  </div>
<template>
```

Good: 
```html
<template>
<div>
  <app-some-component @click="normalize"></app-some-component>
  {{wordmangler}}
</div>
</template>

// component
<style>
Vue.component({
  computed: {
    wordmangler: () => this.fullName.split(' ').map(function (word) {
        return word[0].toUpperCase() + word.slice(1)
      }).join(' ')
  },
  methods: {
    normalize: (event) => event.target.value.reduce((pv, cv) => {
      pv[cv.id] = cv;
      return pv; 
    }, {})
  }
})
</style>
```

Better: 
```html
// template
<template>
  <app-some-component @click="normalize">
    {{wordmangler}}
  </app-some-component>
</template>

// javascript
<script>
const normalizeData = (event) => event.target.value.reduce((pv, cv) => {
    pv[cv.id] = cv;
    return pv; 
  }, {})

const mangleWord = (val) => val.split(' ').map(function (word) {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' '),

// component

Vue.component({
  computed: {
    wordmangler: () => mangleWord(this.fullName),
  }, 
    normalize: (val) => normalizeData(val), 
  methods: {
  }
})
</script>
```

* Don't mutate props *directly* in a child component. They can still be mutated, but they should use the .sync/emit pattern. 

With .sync/emit (and required props), you can allow even your dumb components to have interactivity while still allowing parent components to dispatch actions. 

Bad
```javascript
// template
Vue.component('TodoItem', {
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  methods: {
    removeTodo () {
      var vm = this
      vm.$parent.todos = vm.$parent.todos.filter(function (todo) {
        return todo.id !== vm.todo.id
      })
    }
  },
  template: `
    <span>
      {{ todo.text }}
      <button @click="removeTodo">
        X
      </button>
    </span>
  `
})
```

Good
```javascript
Vue.component('TodoItem', {
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  template: `
    <span>
      {{ todo.text }}
      <button @click="$emit('delete')">
        X
      </button>
    </span>
  `
})
```