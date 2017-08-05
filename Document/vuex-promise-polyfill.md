# Vuex Promise 폴리필(Polyfill)

[es6-promise 모듈](https://github.com/stefanpenner/es6-promise)을 설치합니다.

```sh
npm install es6-promise
```

자동으로 폴리필을 적용하려면 다음과 같이 사용합니다.

```js
import Primise from 'es6-promise';
Primise.polyfill();
```

또는 아래와 같이 단축 사용도 가능합니다.

```js
import 'es6-promise/auto';
```

설정 방법은 다음을 참고하세요.

```js
// src/main.js
import 'es6-promise/auto';

import Vue      from 'vue';
import App      from './App';
import router   from './router';
import store    from './store';
import { sync } from 'vuex-router-sync';
import VueHead  from 'vue-head';

sync(store, router);
Vue.use(VueHead);

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
```

<!-- https://github.com/vuejs-templates/webpack/issues/474 -->