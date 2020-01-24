work in progress :construction:

## Install
```
yarn add postwoman-sfc
or
npm i -S postwoman-sfc
```

## Usage
You can declare it globally (in src/main.js)

```javascript
import PwRest from 'postwoman-sfc'
Vue.use(PwRest)
```

or locally (inside a component)
```javascript
import { PwRest } from 'postwoman-sfc'

export default {
  components: { PwRest }
}
```

And use it
```xml
<pw-rest
  :description="'A list of users'"
  :path="'api/users'"
  :url="'https://reqres.in/'"
/>
```
