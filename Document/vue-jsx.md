# VueJS 템플릿 대신 JSX 사용하기

VueJS는 템플릿(Template) 방식을 지원하지만, 일부 사용자는 React의 JSX를 사용하고 싶어합니다. 템플릿 방식이 일반적으로 편리하고 사용법이 쉽습니다.
반면 JSX는 템플릿 방식보다 유연하죠. 문제는 VueJS의 장점 중 일부인 양방향 데이터 바인딩(`v-model`), 이벤트 수식어(`Event Modifiers`)를 JSX에서는 지원하지 않는다는 점입니다.

하지만 JSX를 반드시 써야하는 상황일 경우, 몇몇 플러그인을 통해 JSX에서도 `v-model`, `Event Modifiers`를 사용할 수 있습니다.

## VueJS 앱에서 JSX 사용하기

먼저 [템플릿 방식을 대체하는 JSX 방식을 사용하는 방법](https://github.com/vuejs/babel-plugin-transform-vue-jsx)부터 살펴봅시다.

### Babel 개발 모듈 설치

NPM 인스톨(install) 명령을 사용하여 다음 개발 모듈을 설치합니다.

- babel-plugin-syntax-jsx
- babel-plugin-transform-vue-jsx
- babel-helper-vue-jsx-merge-props
- babel-preset-env

설치 구문은 다음과 같습니다.

```js
npm install\
  babel-plugin-syntax-jsx\
  babel-plugin-transform-vue-jsx\
  babel-helper-vue-jsx-merge-props\
  babel-preset-env\
  --save-dev
```

그리고 `.babelrc` 파일에 다음 구문을 추가합니다.

```json
{
  "presets": ["env"],
  "plugins": ["transform-vue-jsx"]
}
```

설치 및 설정이 마무리 되면 JSX를 사용할 수 있게됩니다.

```html
<div id="jsx-syntax">{this.text}</div>
```

위 방법은 Virtual DOM 방식으로 구현된 아래 코등와 동일하게 동작합니다.

```js
h('div', {
  attrs: {
    id: 'jsx-syntax'
  }
}, [this.text])
```

`h` 함수는 Vue 인스턴스 `$createElement()` 메서드의 별칭입니다. 이는 반드시 `render()` 함수 내에서만 사용할 수 있습니다.
아래 구문을 참고하세요.

```js
Vue.component('jsx-example', {
  render (h) { // <-- h 매개변수는 render() 함수 내부에 존재해야 합니다.
    return <div id="jsx-syntax">{this.text}</div>
  }
})
```

만약 `babel-plugin-transform-vue-jsx` v3.4 이상을 사용 중이라면 `h` 함수를 생략할 수 있습니다.
이유는 v3.4 버전부터 자동으로 `const h = this.$createEleemnt`가 주입되기 때문입니다.

```js
Vue.component('jsx-example', {
  // 렌더
  render () { // h 모듈이 자동 주입됩니다.
    return <div id="jsx-syntax">{this.text}</div>
  },
  // 메서드
  myMethod: function () { // h 모듈이 자동 주입되지 않습니다.
    return <div id="jsx-syntax">{this.text}</div>
  },
  someOtherMethod: () => { // h 모듈이 자동 주입되지 않습니다.
    return <div id="jsx-syntax">{this.text}</div>
  }
})

// TypeScript
@Component
class App extends Vue {
  get computed () { // h 모듈이 자동 주입됩니다.
    return <div id="jsx-syntax">{this.text}</div>
  }
}
```

### React JSX와 차이점은?

VueJS v2.0에서 구현된 vnode 형식은 React와 다릅니다. `createElement` 함수의 두번째 인자는 데이터 객체(data object) 입니다.

```js
render (h) {
  return h('div', {
    // 컴포넌트 속성
    props: {
      msg: 'hi'
    },
    // 일반 HTML 속성
    attrs: {
      id: 'foo'
    },
    // DOM 속성
    domProps: {
      innerHTML: 'bar'
    },
    // 이벤트 핸들러는 "on" 속성 내에 중첩되어 있습니다.
    // v-on : keyup.enter와 같은 수식어는 지원되지 않습니다.
    // 핸들러의 keyCode를 사용하여 수동으로 구현은 가능합니다.
    on: {
      click: this.clickHandler
    },
    // 컴포넌트 전용 속성으로 컴포넌트 vm.$emit에 의해
    // 네이티브 이벤트가 수신되는 것을 허용합니다.
    nativeOn: {
      click: this.nativeClickHandler
    },
    // class 속성은 특별한 모듈로 `v-bind:class`와 동일합니다.
    class: {
      foo: true,
      bar: false
    },
    // style 속성은 `v-bind:style`와 같습니다.
    style: {
      color: 'red',
      fontSize: '14px'
    },
    // 다른 특별한 속성들
    key: 'key',
    ref: 'ref',
    // ref는 v-for 디렉티브를 사용하는
    // elements/components에 사용됩니다.
    refInFor: true,
    slot: 'slot'
  })
}
```

VueJS v2 이상 버전에서는 위 코드를 아래 코드처럼 바꿀 수 있습니다.

```js
render (h) {
  return (
    <div
      // 일반 속성 또는 컴포넌트 속성은 다음과 같이 작성합니다.
      id="foo"
      // DOM 속성의 경우, `domProps` 접두사를 사용해야 합니다.
      domPropsInnerHTML="bar"
      // 이벤트 리스너의 경우, `on` 또는 `nativeOn` 접두사를 사용해야 합니다.
      onClick={this.clickHandler}
      nativeOnClick={this.nativeClickHandler}
      // 특별한 속성들은 다음과 같이 작성합니다.
      class={{ foo: true, bar: false }}
      style={{ color: 'red', fontSize: '14px' }}
      key="key"
      ref="ref"
      // ref는 v-for 디렉티브를 사용하는
      // elements/components에 사용됩니다.
      refInFor
      slot="slot">
    </div>
  )
}
```

### 컴포넌트 Tip

사용자 정의 요소(Custom Element)가 소문자로 시작할 경우, 문자열 ID로 처리되고 등록된 컴포넌트를 조회합니다.
반면 대문자로 시작할 경우, 식별자로 처리되어 바로 사용할 수 있습니다. 아래 예시를 참고하세요.

```js
import Todo from './Todo.js'

export default {
  render (h) {
    return <Todo/> // Todo 컴포넌트 옵션을 등록할 필요가 없습니다.
  }
}
```

### JSX 스프레드(Spread)

JSX 스프레드가 지원됩니다. 플러그인은 중첩된 데이터 속성을 스마트하게 병합합니다.

```js
const data = {
  class: ['b', 'c']
}
const vnode = <div class="a" {...data}/>
```

병합된 데이터는 아래와 같이 처리됩니다.

```js
{ class: ['a', 'b', 'c'] }
```

### Vue 디렉티브(Directives)

**JSX를 사용할 경우, VueJS의 디렉티브는 거의 사용할 수 없습니다.** 유일하게 `v-show={value}` 구문은 사용할 수 있습니다.
다른 디렉티브는 프로그래밍 방식으로 직접 구현하여야 합니다. 예를 들어 `v-if`는 3항식을 사용하고, `v-for`는 `[].map()` 식을 사용합니다.

사용자 정의 디렉티브를 만들 경우, `v-name={value}`로 사용할 수 있습니다. 다만 VueJS 디렉티브 작성 시 지원하는 전달인자(directive arguments), 수식어(modifiers)는 지원되지 않습니다.
이 문제를 해결하기 위한 방법은 다음과 같습니다. `value`를 포함한 객체의 속성을 통해 전달하는 것입니다. 아래처럼 말이죠.

```js
const directives = [
  { name: 'my-dir', value: 123, modifiers: { abc: true } }
]

return <div {...{ directives }}/>
```

---

## JSX에서 `v-model` 디렉티브를 사용하려면?

혹자는 `v-model`에 문제가 있다고 지적하기도 하지만, 컴포넌트를 제작 시 코드의 양을 줄여줄 수 있어 매우 유용합니다.
`babel-plugin-jsx-v-model` 플러그인을 설치하면 `v-model` 디렉티브를 JSX에서 사용할 수 있게 됩니다.

```sh
# babel-plugin-jsx-v-model 개발 모듈을 설치합니다.
npm install babel-plugin-jsx-v-model --save-dev
# Yarn 을 사용해 설치할 수도 있습니다.
yarn add babel-plugin-jsx-v-model -D
```

`.babelrc` 문서에는 다음과 같이 추가합니다.

```js
"plugins": ["jsx-v-model", "transform-vue-jsx"]
```

사용법은 다음과 같습니다.

```js
export default {

  // 데이터
  data: () => ({
    syncData: ''
  }),

  // JSX
  render: h => {
    return (
      <div class="component-wrapper">
        <h2>{this.syncData}</h2>
        // 인풋 요소에 syncData 속성을 양방향 데이터딩바인딩 합니다.
        <input type="text" v-model={this.syncData} />
      </div>
    )
  }

}
```

`v-model`을 사용하지 않을 경우, 아래와 같이 `onInput`, `value`을 매번 사용하여 코드를 작성해야 합니다.

```js
export default {

  data: () => ({
    syncData: ''
  }),

  render: h => {
    return (
      <div class="component-wrapper">
        <h2>{this.syncData}</h2>
        <input type="text" value={this.syncData} onInput={this.setSyncData}/>
      </div>
    )
  },

  methods: {
    setSyncData(evt) {
      this.syncData = evt.target.value
    }
  }

}
```

## JSX 이벤트 수식어(Event modifiers)

이벤트 수식어는 VueJS가 지원하는 깔끔한 방식이지만, 템플릿이 아닌 JSX에서는 기본 지원되지 않습니다.
하지만 `babel-plugin-jsx-event-modifiers` 플러그인을 사용하면 이를 사용할 수 있게 됩니다.
(이 플러그인을 사용하면 React JSX에서도 이벤트 수식어를 사용 할 수 있게 됩니다)

하지만 VueJS 만큼 다양한 이벤트 수식어를 현재 지원하고 있지 않기 때문에 필요한 경우 직접 추가해서 사용해야 합니다.
이 플러그인은 단지 표준 버블링 컨트롤(standard bubbling control)과 키 수식어(key modifiers)만 제공할 뿐입니다.

- `:stop` === `event.stopPropagation()` 래핑
- `:prevent` === `event.preventDefault()` 래핑
- `:k[key code]` === 일치하는 키코드가 눌려진 경우에만 이벤트가 동작합니다. (Vue 구현과는 다릅니다, 키 이벤트에만 유효)
- `:{key name}` === 일치하는 키이름이 눌려지면 이벤트가 동작합니다.
  - `esc`
  - `tab`
  - `enter`
  - `space`
  - `up`
  - `down`
  - `left`
  - `right`
  - `delete`

이 플러그인을 설치하려면 다음의 과정을 진행해보세요.

```sh
# babel-plugin-jsx-event-modifiers 개발 모듈을 설치합니다.
npm install babel-plugin-jsx-event-modifiers --save-dev
# Yarn 을 사용해 설치할 수도 있습니다.
yarn add babel-plugin-jsx-event-modifiers -D
```

`.babelrc` 파일에 아래와 같은 설정 내용을 추가합니다.

```json
"plugins": ["jsx-event-modifiers", "transform-vue-jsx"]
```

사용법은 다음과 같습니다.

```js
export default {
  render: h => {
    return (
      <div class="component-wrapper">
        // :prevet 수식어 사용
        <button onClick:prevent={this.doSomething}>
          Do Something
        </button>
        // :enter 수식어 사용
        <input type="text"
          placeholder="To do something, press enter."
          onKeyUp:enter={this.doSomething}
        />
      </div>
    )
  },

  methods: {
    doSomething() {
      ...
    }
  }
}
```

수식어를 사용할 수 없었다면 아래와 같이 작성해야 합니다.

```js
export default {
  render: h => {
    return (
      <div class="component-wrapper">
        <button onClick={evt => {
          // evt 객체를 받아 preventDefault() 합니다.
          evt.preventDefault();
          this.doSomething();
        }}>
          Do Something
        </button>

        <input type="text"
          placeholder="To do something, press enter."
          onKeyUp={evt => {
            // evt 객체를 받아 charCode 값을 비교하여 처리합니다.
            if (evt.charCode === 13) {
                this.doSomething()
            }
          }}
        />
      </div>
    )
  },

  methods: {
    doSomething() {
      ...
    }
  }
}
```


