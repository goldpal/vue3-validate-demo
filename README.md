# Validate forms

You know what they say, you can not trust a thief or a murderer. You know who you can trust even less? That's right, the u s e r 😤. Therefore we need to strictly watch their every step and every action. Especially when we ask THEM (dear god) to input something in a form and send it to us (YIKES).

## Example + Setup

1. Make sure you clone this repository or install it with npm

    ```cmd
    npm i -S git+https://github.com/goldpal/vue3-validate-demo.git
    ```

2. Set up your component

    ```js
    import {
      minLength,
      required,
      type,
      useValidation,
      validateIf
    } from '@goldpal/validate'
    import { computed, reactive } from 'vue'

    // Create your form
    const form = reactive({
      value: [1, 2, 3],
      info: {
        contact: 'the_value_should_be_an_email'
      }
    })

    // Create rules your form must conform to
    const rules = {
      // Rule keys must match the keys of your form
      value: {
        // Value is required
        required,
        // Using a helper, we can add conditional validations
        // Here we check if length of value is larger than 1 but ONLY if the value is an array
        shouldCheckType: validateIf(
          () => Array.isArray(form.value),
          minLength(1)
        )
      },
      // You can also nest rules
      info: {
        contact: {
          required,
          email
        }
      }
    }

    // Create your validation instance
    const { validate, errors, reset } = useValidation(form, rules)
    ```

3. Trigger form validation whenever and wherever you want

    ```js
    async function onSubmit(e) {
      e.preventDefault()

      validate()
        // ctx is an object which contains information about validation checks
        .then((ctx) => {
          /* Executes if validation passes */
    
        })
        .catch((ctx) => {
          /* Executes if validation fails */

          // Extract a rule property for its information
          const { info } = ctx

          info.contact.id // 'contact'
          info.contact.value // 'the_value_should_be_an_email'
          info.contact.invalid // 'true'
          info.contact.errors.email // 'Value must be in a valid email format'
    
        })
    }

    // You can also write the same function like this

    async function onSubmit(e) {
      e.preventDefault()

      try {
        await validate()
      }
      catch (errors: any) {

        // Handle errors or reset form
        // Resetting form clears all errors though, so it is recommended to display
        // them in some manner and reset them when user begins using the form again
        reset()
      }
    }
    ```

## API

### `useValidation(form, rules, options)`

Main composable which is used to initiate form validation context as well as return validation methods.

- `form` reactive form object
- `rules` object or a computedRef object
- `options`
  - `autoclear` (default: false) resets all errors the first time a form is updated after `validation()` was ran
  - `proactive` (default: false) runs form validation on every input

Returns

- `validate(): Promise<State>` performs a form validation and resolves with the current State of the form
- `reset(): void` clears all errors after form validation
- `errors` an object which contains all form fields and their status
- `state` an object which includes state of the entire form

---

## Validators

Collection of validators used to construct rule objects

### `required`

Requires non-empty value.

```js
const rules = {
  name: { required }
}
```

---

### `minLength(min)`

Requires input to have a minimal specified length.

- **Parameters**
  - `Ref<number> | number` min
- **Works with**
  - `string`, `Set<any>`, `Map<any, any>`, `any[]` and `Object`

```js
const rules = {
  name: { minLength: minLength(10) }
}
```

---

### `maxLength(max)`

Requires input to have a maximum specified length.

- **Parameters**
  - `Ref<number> | number` max
- **Works with**
  - `string`, `Set<any>`, `Map<any, any>`, `any[]`, `Object`

```js
const rules = {
  name: { minLength: minLength(10) }
}
```

---

### `minLenNoSpace`

Input must be a string and excluding spaces must be equal or greater to the provided min value.

- **Parameters**
  - `Ref<number> | number` min - Minimum allowed string length
- **Works with**
  - `string`

```js
const rules = {
  value: {
    // A B C D E F => false
    // A B C D E F G => true
    min: minLenNoSpace(7)
  }
}
```

---

### `maxLenNoSpace`

Input must be a string and excluding spaces must be equal or lesser to the provided max value.

- **Parameters**
  - `Ref<number> | number` max - Maximum allowed string length
- **Works with**
  - `string`

```js
const rules = {
  value: {
    // H  e  l  l  0  => true
    // Bummer => false
    max: maxLenNoSpace(5)
  }
}
```

---

### `between(min, max)`

Requires input to be a number or Date within the specified bounds.

- **Parameters**
  - `Ref<number> | number | Date` min
  - `Ref<number> | number | Date` max
- **Works with**
  - `number`, `Date`

```js
const rules = {
  numberVal: { between: between(0, 10) },
  dateVal: {
    between: between(new Date('12/24/2020'), new Date('12/24/2022').getTime())
  }
}
```

---

### `url`

Requires input to be a valid URL.

---

### `email`

Requires input to be a valid email address.

---

### `decimal`

Requires input to be a number and contain decimals.

---

### `hasSpecialChars`

Checks wether an input contains any special characters

---

### `sameAs(value, lenient)`

Requires input to be the same as the provided value in the rule.

- **Parameters**
  - `Ref<any> | any` value
  - `boolean` (default: false) lenient - wether to only compare based on value or value and type (eg. `==` vs `===`)
- **Works with**
  - `any`

```js
const rules = {
  name: { sameAs: sameAs('Hello world') }
}
```

---

### `match(regex)`

Requires value to match the provided regular expression check.

- **Parameters**
  - `string | RegExp` regex
- **Works with**
  - `string`

```js
// Checks wether string contains only letters and numbers
const rules = {
  name: {
    noSpecialChars: match(/[^a-zA-Z0-9]/)
  }
}
```

---

### `contains(items, split)`

Checks wether string input contains certain words or characters. If you write multiple words. It will check against each word individually.

- **Parameters**
  - `string | string[]` items - Words to check for
  - `boolean` (default: false) exact - Wether to match against the entire input string or split it by spaces. Default to true
- **Works With**
  - `string`

```js
const rules = {
  value: {
    // Input: 'Hello Dear World' => false
    containsExact: contains('Hello World', true),
    // Input: 'Hello World' => true
    containsSome: contains('Hello Dear World')
  }
}
```

---

### `startsWith(str, pos)` & `endsWith(str, pos)`

Checks wether the input value is a string and starts or ends with the provided parameter.
You can optionally specify the position where the check begins in the input value

- **Parameters**
  - `string | Ref<string>` String we are matching against
  - `number` Starting position of the matching
    - *Note*: When using `endsWith`, the position starts at the end and goes backwards
- **Works with**
  - `string`


```js
const rules = {
  value: {
    // Input 'Hello World' => true
    start: startsWith('llo', 2),
    // Input 'Hello World' => false
    endsBad: endsWith('Hello')
  }
}
```

---

### `type[typeToCheckFor]`

This is a special object rule in which you need to specify the type you want to check against.
Note, each type check is also available as a standalone import without using the `type` object. Only difference is
that you need to prefix it.

```js
// This import method
import { isMap, isStr } from 'v-valid'
// Equals to this one
import { type } from 'v-valid'
const { str, map } = type

```

- **Available type checks**
  - `type.str` - requires value to be an instance of `string`
  - `type.num` - requires value to be an instance of `number`
  - `type.arr` - requires value to be an instance of `Array`
  - `type.obj` - requires value to be an instance of `Object`
  - `type.set` - requires value to be an instance of `Set`
  - `type.map` - requires value to be an instance of `Map`
  - `type.date` - requires value to be an instance of `Date`
  - `type.symbol` - requires value to be an instance of `Symbol`

```js
const rules = {
  name: { array: type.arr }
}
```

## Helpers

### `withLabel(label, rule)`

Allows you to add a custom error message to ant validation rule.

- **Parameters**
  - `string | Label` label
  - `ValidationRule | ValidationRuleObject` rule

```js
const rules = {
  name: {
    required: withLabel('You MUST fill this input!!', required)
  }
}
```

We can also use a method instead of a string when creating custom label. This method receives the validated `value` as the first parameter and all the subsequent parameters from the validation rule. If there are any.

```js
const rules = {
  name: {
    between: withLabel(
      (value, min, max) =>
        `The value (${value}) must fit between ${min} and ${max}`,
      // If error occurs, the message will read like this:
      // 'The value (<value>) must fit between 5 and 10'
      between(5, 10)
    )
  }
}
```

---

### `and(rules)`, `or(rules)`

Used when we want to apply multiple rules together into one.

- `and(...ValidationRule[])` Requires all rules to be passing to resolve
- `or(...ValidationRule[])` Requires at least one rule to be passing to resolve

```js
const rules = {
  name: {
    // We want the value to be an array with at least 10 items in it
    group: and(type.arr, minLength(10))
  }
}
```

### `not(rules)`

Used in situations where we want to invert a rule results. Can also be used with multiple rules, only resolving when all of them are failing.

```js
const rules = {
  name: {
    // We want the input value to be anything but a string
    notString: not(type.str)
  }
}
```

---

### `validateIf(condition, rule)`, `validateIfNot(condition, rule)`

Performs validation if the provided condition is met. Due to implementation limits, you can't use a rule as a condition. For that please use the `and` helper.

- **Parameters**
  - `boolean | () => boolean | Ref<boolean> | Promise<boolean>` condition
  - `ValidationRule` rule

```js
const rules = {
  name: {
    // We want to check min length of value but only if value is an array
    minLength: validateIf(Array.isArray(form.value), minLength(5)),

    // If value is not a number, required it to match "Hello World"
    sameAs: validateIfNot(typeof form.value === 'number', sameAs('Hello World'))
  }
}
```

---

### `$def(rule, message): ValidationRule`

Helpers used to create validation rules

- **Parameters**
  - `(value: any, args: Record<string, unknown>) => boolean | Promise<boolean>` rule
  - `string | Label | undefined` label

```js
// Define rule without any parameters
const required = $def(value => value !== undefined && value !== null, 'Value is required')

// Defining rules with parameters
// Rule which requires value to be an array and be at least n length
const arrAndMinLen = $defParam < { length: number } > (
  (value, { length }) => isArray(value) && value.length >= length,
  (_, { length }) => `Array with at least ${length} length`
)

// Asynchronous rule, the main use case here is validating
// login passwords or other API related things
const checkPassword = $def(
  password =>
    new Promise((resolve) => {
      const result = await someApiCall(password)
      resolve(result)
    }),
  'Incorrect password. Please try again.'
)
```

---

### `$test(rule, value): Promise<boolean> | boolean`

Perform a synthetic validation outside of any validation scope.

- **Parameters**
  - `ValidationRule` rule
  - `value` any

```js
const result = $test(minLength(5), [1, 2, 3]) // False
const result = $test(maxLength(5), [1, 2, 3]) // True
const result = $test(type.symbol, Symbol('Symbol')) // True

// With `and`, `not`, `or` and any async custom validators we have to await the returned promise
const result = await $test(or(type.str, type.num), 'Hello World') // True
```
