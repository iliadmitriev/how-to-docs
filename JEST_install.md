1. Install Jest, vue-jest and @vue/test-utils
```bash
npm install --save-dev jest vue-jest @vue/test-utils
```

2. Add run script to `package.json` file (merge with exiting lines)
```json
{
  "scripts": {
    "test": "jest"
  }
}
```

3. Babel

Install modules

```bash
npm install --save-dev babel-core babel-jest \
                      @babel/core @babel/preset-env \
                      @babel/plugin-transform-runtime \
                      @babel/runtime
```

Modify `.babelrc` file

```json
{
  "presets": [
    "@babel/preset-env"
  ],
  "env": {
    "test": {
      "plugins": ["@babel/plugin-transform-runtime"]
    }
  },
  "plugins": [
    ["@babel/transform-runtime"]
  ]
}
```

4. CSS, LESS, SASS files include (stub)

Install `jest-transform-stub`

```bash
npm install --save-dev jest-transform-stub
```

5. create jest config file `jest.config.js`
  * turn on verbosity
  * specify file extentions
  * add transform for vue and babel
  * add stub for css
  * ignore paths
  * enable coverage
  * coverage threshould
  * jsdom default test environment
 
```javascript
module.exports = {

  verbose: true,

  testEnvironment: 'jsdom',

  moduleFileExtensions: [
    'js',
    'json',
    'vue'
  ],

  moduleNameMapper: {
    "^.+\\.(css|styl|less|sass|scss|png|jpg|ttf|woff|woff2)$": "jest-transform-stub",
    // enable import beginning with @/ - as reference to <root>/src/ folder
    "^@/(.*)$": "<rootDir>/src/$1"
  },

  modulePathIgnorePatterns: [
    '<rootDir>/build/',
    '<rootDir>/dist/',
    '<rootDir>/coverage/'
  ],

  transform: {
    '.*\\.(vue)$': 'vue-jest',
    '.*\\.(js)$': 'babel-jest'
  },

  collectCoverage: true,
  collectCoverageFrom: ['src/**/*.{js,vue}', '!**/node_modules/**'],
  coveragePathIgnorePatterns: [],
  coverageThreshold: {
    global: {
      branches: 100,
      functions: 100,
      lines: 100,
      statements: 100
    }
  },

}

```

6. Setup vuetify

```javascript
import {createLocalVue} from "@vue/test-utils";
import Vuetify from "vuetify";
import Component from "@/components/Component.vue";

// vuetify hack
import Vue from "vue";
Vue.use(Vuetify)

// vuetify instance
vuetify = new Vuetify()

// create localVue instance with vuetify
localVue = createLocalVue()

// mount component which uses vuetify
const wrapper = mount(Component, {
      localVue,
      vuetify
    })


```

Vuetify fix `SyntaxError: Unexpected token 'export'`
add to config file `jest.config.js`

```javascript
module.exports = {
// ...

  transformIgnorePatterns: [
    'node_modules/(?!vuetify)'
  ],
// ...
}
```

7. Set up a list of modules in the suite which is executed before each test file

create file `tests/setup.js`

```javascript
import {createLocalVue} from "@vue/test-utils";
import Vuex from "vuex";

// localVue variable
global.localVue = createLocalVue()
localVue.use(Vuex)

// fetch function mock
global.fetch = jest.fn()

// localStorage.getItem mock
Storage.prototype.getItem = jest.fn()

// localStorage.setItem mock
Storage.prototype.setItem = jest.fn()

```

add to config file `jest.config.js`

```javascript
module.exports = {
//...

  setupFilesAfterEnv: ["<rootDir>/tests/setup.js"],

//...
}
```
