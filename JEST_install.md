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
npm install --save-dev babel-core babel-jest @babel/core @babel/preset-env @babel/plugin-transform-runtime
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
  }
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
