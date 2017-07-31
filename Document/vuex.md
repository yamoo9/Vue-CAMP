# Vuex 중앙 저장소 데이터 관리 패턴 라이브러리

## [Getting started with Vuex state management for VueJS](https://blog.pusher.com/getting-started-vuex-state-management-vuejs/)

이 게시물은 [Pusher Guest Writer program](https://pusher.com/guest-writer-program)에 의해 작성된 내용을 번역(의역)한 글입니다.

VueJS는 인터랙티브 웹 인터페이스를 구축하기 위한 라이브러리로 간단하고 유연하게 사용 가능한 데이터-반응성 컴포넌트 API를 제공합니다. 다수의 개발자들이 VueJS를 사용하는 이유는 가볍고, 쉽기 때문입니다.

쉽다고는 하지만 컴포넌트 간 통신을 위해 부모/자식 컴포넌트 간 `props`를 전달하거나, 변경해야하는 문제가 발생합니다. 종종 `props`를 사용하지 않고 `this.$parent.data`와 같은 접근 방식을 사용할 수 있습니다. 자식 컴포넌트는 부모 컴포넌트에 `$emit()`을 사용하여 이벤트를 발신하는 방법으로 통신합니다.

이러한 방법이 부모 컴포넌트의 데이터를 변경하는 간단한 방법일지 모르지만, 프로젝트 규모가 커지고 컴포넌트가 많아짐에 따라 받는 스트레스 또한 가중됩니다. 예를 들어 30개의 컴포넌트가 존재할 경우, 각 30개의 다른 이벤트가 발생하게 됩니다. 오류가 발생할 확률 또한 커질 뿐더러 디버깅에 많은 시간을 투자해야하는 문제가 발생합니다.

이러한 문제를 해결할 방법은 없을까요?

**Vuex**. Vuex를 사용하면 효율적으로 애플리케이션의 상태 관리를 할 수 있습니다. Vuex는 Vue 애플리케이션에서 Flux와 유사한 애플리케이션 아키텍처를 구현하는데 도움을 주는 라이브러리입니다.

### Vuex란?

Vuex 공식 문서에서는 VueJS 애플리케이션을 위한 상태 관리 패턴 라이브러리라고 소개합니다. 상태(`state`)를 예측 가능한 방식(디버깅 용이)으로 관리하며, 게터(`getters`), 뮤테이션(`mutations`), 액션(`actions`) 등 인터페이스를 제공하여 애플리케이션의 모든 컴포넌트 사이 통신을 책임지는 중앙 집중식 저장소(`Store`) 역할을 합니다.

#### store

Vuex에서 생성 된 객체를 **저장소(`store`)** 라고 합니다.

```js
const store = new Vuex.Store();
```

#### state

상태(`state`)는 저장소(`store`)에서 사용하려는 데이터가 포함된 JavaScript 객체를 나타냅니다. 이는 Vue 컴포넌트의 데이터(`data`) 속성과 같습니다.

```js
const store = new Vuex.Store({
  state: {}
});
```

> Vuex는 단 하나의 상태 트리(Single State Tree)를 사용합니다. 이 객체는 모든 애플리케이션의 상태를 포함하며 관리됩니다. 요컨대 각 애플리케이션에서 하나의 저장소만 사용해야 한다는 것입니다. 단 하나의 상의 트리를 사용하면 특정 상태를 쉽게 찾을 수 있고, 디버깅을 위해 현재 애플리케이션 상태의 스냅 샷을 만들 수도 있습니다.

#### getters

상태(`state`)를 읽어오는 역할을 수행하는 헬퍼 함수입니다. 저장소의 계산된 속성(computed properties)으로 생각하면 좋을 것 같습니다. 계산된 속성과 마찬가지로 getters의 반환 값은 종속성에 따라 캐시되며 종속성을 가진 속성이 변경되면 다시 처리하여 변경된 값을 반환합니다.

```js
const store = new Vuex.Store({
  state: {},
  getters: {}
});
```

#### mutations

뮤테이션은 상태를 변경할 수 있는 헬퍼 함수로, Vuex 저장소 시스템에서 상태를 바꿀 수 있는 유일한 수단입니다. Vuex 뮤테이션은 이벤트와 같습니다. 각 뮤테이션은 식별 가능한 문자(식별자)와 핸들러를 가집니다. 핸들러는 상태를 수정하는 로직을 가지며, 첫번째 인자로 상태(`state`)를 전달 받습니다.

```js
const store = new Vuex.Store({
  state: {},
  getters: {},
  mutations: {
    MUTATION_STATE(state){
      // state는 Vuex 저장소의 데이터
    }
  }
});
```

#### actions

액션은 뮤테이션을 실행하는 함수를 말합니다. 뮤테이션과 달리 비동기(Async) 처리가 가능합니다. Vuex v2 부터는 액션을 디스패치(`dispatch`)할 수 있습니다. 액션은 VueJS 컴포넌트의 메서드(`methods`)와 유사합니다.

```js
const store = new Vuex.Store({
  state: {},
  getters: {},
  mutations: {
    MUTATION_STATE(state, payload){
      // state는 Vuex 저장소의 데이터
      // payload는 전달 받은 데이터(수정을 위한)
    }
  },
  actions: {
    mutation_state(store, payload){
      // store는 저장소 객체
      // {commit}을 사용하여 commit 함수만 전달받는 것도 가능하다.
      // commit() 함수는 뮤테이션을 실행시킨다.
    }
  }
});
```

## Vuex 실습을 위한 애플리케이션

TodoList 앱을 제작해보면서 Vuex를 실습해 봅시다. 다음은 완성된 결과물입니다.

![](https://blog.pusher.com/wp-content/uploads/2017/07/getting-started-vuex-state-management-vuejs-example.gif)

### 1. 애플리케이션 스캐폴딩

vue-cli를 글로벌 설치한 후, 템플릿을 사용하여 프로젝트를 스캐폴딩합니다. 생성된 프로젝트 디렉토리로 이동한 후, 필요한 개발 모듈을 설치합니다.

```js
$ npm install --global vue-cli
$ vue init yamoo9/vue-simple todoList
$ cd todoList
$ npm install
```

### 2. Vuex 설치

vuex 개발 모듈을 프로젝트에 설치합니다.

```js
$ npm install --save-dev vuex
```

### 3. Vuex에서 사용될 객체 설정

#### 3.1 state

저장소에서 관리할 상태 객체로 `todos` 속성을 가지며 초기 데이터로 빈 배열을 설정합니다.

```js
state: {
  todos: []
}
```

#### 3.2 getters

저장소 상태를 읽어오는(가져오는) 함수로 `not_done`, `done`을 설정합니다.

- `not_done`: `todos` 상태의 데이터 집합에서 `todo.status` 값이 `false`로 설정되어 있는 데이터 셋을 필터링하여 반환합니다.
- `done`: `todos` 상태의 데이터 집합에서 `todo.status` 값이 `true`로 설정되어 있는 데이터 셋을 필터링하여 반환합니다.

```js
getters: {
  not_done(state){
    return state.todos.filter(todo => todo.status === false);
  },
  done(state){
    return state.todos.filter(todo => todo.status === true);
  }
}
```

#### 3.3 mutations

저장소 상태를 변경할 수 있는 함수를 추가합니다.

- `ADD_TODO`: 새로운 `todo` 객체를 `todos` 저장소의 상태에 추가합니다.
- `COMPLETE_TODO`: 할 일(`todo`)이 완료되었음을 처리하는 함수입니다.

```js
mutations: {
  ADD_TODO(state, new_todo){
    state.todos.push(new_todo);
  },
  COMPLETE_TODO(state, todo){
    state.todos.find(x => x.todo === todo).status = true;
  }
}
```

#### 3.4 actions

- `add_todo`: 새로운 할 일을 구성한 후, 뮤테이션에 커밋하는 일을 수행하는 함수입니다.
- `complete_todo`:이 함수는 뮤테이션에 커밋하여 할 일(`todo`) 항목을 완료하도록 설정합니다.

```js
actions: {
  add_todo({commit}, new_todo){
    let set_new = {
      todo: new_todo,
      status: false
    };
    commit('ADD_TODO', set_new);
  },
  complete_todo({commit}, todo){
    commit('COMPLETE_TODO', todo);
  }
}
```

### 4. 설정된 각 객체를 Vuex에 구성

위에서 정의한 각 객체들(`state`, `getters`, `mutations`, `actions`)을 생성된 스토어 객체에 설정해봅니다.

```js
// src/store.js

import Vue from "vue";
// Vuex 모듈 로드
import Vuex from "vuex";
// Vuex 플러그인 사용 설정
Vue.use(Vuex);

// Vuex 저장소 객체 생성
const store = new Vuex.Store({
  // 상태 객체 설정
  state: {
    todos: []
  },
  // 게터(getters) 객체 설정
  getters: {
    not_done(state){
      return state.todos.filter(todo => todo.status === false);
    },
    done(state){
      return state.todos.filter(todo => todo.status === true);
    }
  },
  // 뮤테이션 객체 설정
  mutations: {
    ADD_TODO(state, new_todo){
      state.todos.push(new_todo);
    },
    COMPLETE_TODO(state, todo){
      state.todos.find(x => x.todo === todo).status = true;
    }
  },
  // 액션 객체 설정
  actions: {
    add_todo({commit}, new_todo){
      let set_new = {
        todo: new_todo,
        status: false
      };
      commit('ADD_TODO', set_new);
    },
    complete_todo({commit}, todo){
      commit('COMPLETE_TODO', todo);
    }
  }
});

// store 객체를 모듈의 기본 값으로 내보내기
export default store;
```

### Vue 컴포넌트에 Vuex Store 객체 연결

아래와 같이 Vue 컴포넌트에 `store` 객체를 설정하면 컴포넌트 내부에서 `this.$store`로 Vuex 저장소 객체에 접근할 수 있으며, 설정된 `getters`, `mutations`, `actions`를 사용할 수 있게 됩니다.

```js
// src/main.js

import Vue from 'vue';
import App from './App';
// Vuex Store 객체 모듈 로드
import store from './store';

Vue.config.productionTip = false;

new Vue({
  el: '#app',
  render: h => h(App),
  // store 객체를 Vue 컴포넌트 내부에 설정
  store
});
```

### Vue 컴포넌트

App.vue 파일

```js
<template lang="pug">
#app
  .container
    .row
      .col-md-6
        hello
      .col-md-6
        done
</template>

<script>
import Hello from './components/TodoList';
import Done from './components/Done';

export default {
  name: 'app',
  components: { Hello, Done }
};
</script>
```

TodoList.vue 파일

```js
<template lang="pug">
.todolist.not-done
  h1 Todos
  input.form-control.add-todo(type='text', v-model='holder', placeholder='Add todo')
  button#checkAll.btn.btn-success(:disabled="holder==''", @click='add_todo') Add Todo
  hr
  ul#sortable.list-unstyled(v-if='not_done_todos')
    li.ui-state-default(v-for='todo in not_done_todos')
      .checkbox
        label
          input(type='checkbox', @click='done_todo(todo.todo)', :value='todo.todo', :checked='todo.status')
          | {{todo.todo}}
  .todo-footer(v-if='not_done_todos')
    strong
      span.count-todos {{not_done_todos.length}}
    | Item(s) Left
</template>

<script>
export default {
  name: 'todoList',
  data () {
    return {
      holder: ''
    }
  },
  methods: {
    add_todo: function(){
      this.$store.dispatch('add_todo', this.holder);
      this.holder = '';
    },
    done_todo: function(todo){
      this.$store.dispatch('complete_todo', todo);
    }
  },
  computed: {
    not_done_todos: function(){
      return this.$store.getters.not_done;
    }
  }
};
</script>
```

Done.vue 파일

```js
<template lang="pug">
.done
  .todolist
    h1 Already Done
    ul#done-items.list-unstyled(v-if='done_todos')
      li(v-for='todo in done_todos') {{todo.todo}}
</template>
<script>
  export default {
    computed: {
      done_todos: function() {
        return this.$store.getters.done;
      }
    }
  }
</script>
```

애플리케이션 [실습 예제 전체 코드](https://github.com/samuelayo/getting_started_with_vuex)를 다운로드 받아 확인해보세요.