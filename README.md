# vue-form  

[![Build Status](https://travis-ci.org/fergaldoyle/vue-form.svg?branch=0.2.2)](https://travis-ci.org/fergaldoyle/vue-form)

Form validation for Vue.js 2.0+

### Install

Available through npm as `vue-form`.

``` js
import vueForm from 'vue-form';

// install globally
Vue.use(vueForm);

// or use the mixin
...
mixins: [vueForm.mixin]
...
```

### Usage

Once installed you have access to four components (`vue-form`, `validate`, `form-errors`, `form-error`) for managing form state, validating form fields and displaying validation error messages.

Example

```html
<div id="app">
  <vue-form :state="formstate" @submit.prevent="onSubmit">

    <validate tag="label">
      <span>Name *</span>
      <input v-model="model.name" required name="name" />

      <form-error field="name" error="required">Name is a required field</form-error>
    </validate>

    <validate tag="label">
      <span>Email</span>
      <input v-model="model.email" name="email" type="email" required />

      <form-errors field="email">
        <div slot="required">Email is a required field</div>
        <div slot="email">Email is not valid</div>
      </form-errors>
    </validate>

    <button type="submit">Submit</button>
  </vue-form>
  <pre>{{ formstate }}</pre>
</div>
```

```js
Vue.use(vueForm);

new Vue({
  el: '#app',
  data: {
    formstate: {},
    model: {
      name: '',
      email: 'invalid-email'
    }
  },
  methods: {
    onSubmit: function () {
      if(this.formstate.$invalid) {
        // alert user and exit early
        return;
      }
      // otherwise submit form
    }
  }
});
```

The output of `formstate` will be:
```js
{
  "$dirty": false,
  "$pristine": true,
  "$valid": false,
  "$invalid": true,
  "$submitted": false,
  "$touched": false,
  "$untouched": true,
  "$pending": false,
  "$error": {
    // fields with errors are copied into this object
  },
  "name": {
    "$name": "name",
    "$dirty": false,
    "$pristine": true,
    "$valid": false,
    "$invalid": true,
    "$touched": false,
    "$untouched": true,
    "$pending": false,
    "$error": {
      "required": true
    }
  },
  "email": {
    "$name": "email",
    "$dirty": false,
    "$pristine": true,
    "$valid": false,
    "$invalid": true,
    "$touched": false,
    "$untouched": true,
    "$pending": false,
    "$error": {
      "email": true
    }
  }
}
```

### Displaying errors
Display single errors with `form-error` or multiple errors with `form-errors`.

The `show` prop supports simple expressions which specifiy when erros should be displayed based on the current state of the field, e.g: `$dirty`, `$dirty && $touched`, `$dirty || $touched`

```html
<form-error field="fieldName" error="errorKey" show="$dirty">Error message</form-error>

<form-errors field="fieldName" show="$dirty && $touched">
  <div slot="errorKeyA">Error message A</div>
  <div slot="errorKeyB">Error message B</div>
</form-errors>
```


### Validators

```
type="email"
type="url"
type="number"
required
minlength
maxlength
pattern
min (for type="number")
max (for type="number")

```

You can use static validation attributes or bindings. If it is a binding, the input will be re-validated every binding update meaning you can have inputs which are conditionally required etc.
```html
<!-- static validators -->
<validate>
  <input type="email" name="email" v-model="model.email" required />
</validate>
<validate>
  <input type="text" name="name" v-model="model.name" maxlength="25" minlength="5" />
</validate>

<!-- bound validators -->
<validate>
  <input type="email" name="email" v-model="model.email" :required="isRequired" />
</validate>
<validate>
  <input type="text" name="name" v-model="model.name" :maxlength="maxLen" :minlength="minLen" />
</validate>
```

#### Custom validators
You can register global and local custom validators. 

Global custom validator
```js
vueForm.addValidator('my-custom-validator', function (value, attrValue, vnode) {
  // return true to set input as $valid, false to set as $invalid
  return value === 'custom';
});
```

```html
<validate>
  <input v-model="something" name="something" my-custom-validator />
</validate>
```

Local custom validator
```html
<validate :custom="{customValidator: customValidator}">
  <input v-model="something" name="something" />
</validate>
```

```js
// ...
methods: {
	customValidator: function (value) {
		// return true to set input as $valid, false to set as $invalid
		return value === 'custom';
	}
}
// ...
```

#### Async validators:

Async validators are custom validators which return a Promise. `resolve()` `true` or `false` to set field vadility. 
```js
// ...
methods: {
  customValidator (value) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(value === 'ajax');
      }, 100);
    });
  }
}
// ...
```

Async validator with debounce (example uses lodash debounce)
```js
methods: {
  debounced: _.debounce(function (value, resolve, reject) {
    fetch('https://httpbin.org/get').then(function(response){
      resolve(response.isValid);
    });
  }, 500),
  customValidator (value) {
    return new Promise((resolve, reject) => {
      this.debounced(value, resolve, reject);
    });
  }
}
```

### State classes
As form and input validation states change, state classes are added and removed

Possible form classes:
```
vf-form-dirty, vf-form-pristine, vf-form-valid, vf-form-invalid, vf-form-submitted
```

Possible input classes:
```
vf-dirty, vf-pristine, vf-valid, vf-invalid

// also for every validation error, a class will be added, e.g.
vf-invalid-required, vf-invalid-minlength, vf-invalid-max, etc
```

Input wrappers (e.g. the tag the `validate` component renders) will also get state classes, but with the `container` prefix, e.g.
```
vf-container-dirty, vf-container-pristine, vf-container-valid, vf-container-invalid
```

### Custom components

When writing custom form field components, e.g. `<my-checkbox v-model="foo"></my-checkbox>` you should trigger the `focus` and `blur` events after user interaction either by triggering native dom events on the root node of your component, or emitting Vue events (`this.$emit('focus)`) so the `validate` component can detect and set the `$dirty` and `$touched` states on the field.

### Component props

#### vue-form
* `state` Object on which form state is set

#### validate
* `state` Optional way of passing in the form state. If omitted form state will be found in the $parent
* `custom` Object containing one or many custom validators. `{validatorName: validatorFunction}`
* `tag` String which specifies what element tag should be rendered by the `validate` component, defaults to `span`

#### form-error
* `state` Optional way of passing in the form state. If omitted form state will be found in the $parent
* `field` String which specifies the related field name
* `error` String which specifies the error key which the error should be shown for
* `tag` String, defaults to `span`
* `show`: String, show error dependant on form field state e.g. `$dirty`, `$dirty && $touched`

#### form-errors
* `state` Optional way of passing in the form state. If omitted form state will be found in the $parent
* `field` String which specifies the related field name
* `tag` String, defaults to `div`
* `show`: String, show error dependant on form field state e.g. `$touched`, `$dirty || $touched`

### Config
Set config options using `vueForm.config`, defaults:

```js
{
    formComponent: 'vueForm',
    errorComponent: 'formError',
    errorsComponent: 'formErrors',
    validateComponent: 'validate',
    errorTag: 'span',
    errorsTag: 'div',
    classPrefix: 'vf-',
    dirtyClass: 'dirty',
    pristineClass: 'pristine',
    validClass: 'valid',
    invalidClass: 'invalid',
    submittedClass: 'submitted',
    touchedClass: 'touched',
    untouchedClass: 'untouched',
    pendingClass: 'pending',
    Promise: window.Promise
}
```
