# Learning `trustedTypes`

Just one of the things I'm learning. https://github.com/hchiam/learning

`trustedTypes` lets you block and handle DOM XSS in one spot. Instead of trying to manually find and fix each individual XSS sink. You can also listen for violations with `securitypolicyviolation`. And report violations to a URL with a `report-uri` in the [CSP](https://github.com/hchiam/learning-csp) header. See all of this in the code snippets below.

Demo: https://codepen.io/hchiam/pen/mdYOZOP?editors=0010

Browser support: https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API#browser_compatibility

There's an official polyfill (use the full version for enforcement and throwing): https://github.com/w3c/trusted-types?tab=readme-ov-file#polyfill and here's a quick test demo of that polyfill: https://w3c.github.io/trusted-types/demo/

## PART 1) Turn on `trustedTypes` so vulnerable inputs require `TrustedHTML`

To turn on `trustedTypes`, you need [browser support](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API#browser_compatibility) (or a [polyfill](https://github.com/w3c/trusted-types?tab=readme-ov-file#polyfill)) and [CSP (Content Security Policy)](https://github.com/hchiam/learning-csp):

```html
<head>
  <meta http-equiv="Content-Security-Policy" content="require-trusted-types-for 'script'">
</head>
```

Now all DOM XSS violations are blocked in JavaScript, and vulnerable inputs require `TrustedHTML` for the browser to know to trust them! See the policy objects in PART 2 for how to provide `TrustedHTML`.

### Detect browser support for `trustedTypes` in JS

```js
const supported = window.trustedTypes;
```

### Now trigger a violation in JS to see if it gets blocked

```js
try {
  element.innerHTML = '<img src=x onerror=alert("xss")>'; // not a TrustedHTML object
} catch(e) {
  console.log('\n\nxss:');
  console.error(e);
  alert('Blocked DOM XSS: \n\n' + e);
}
```

and now that all such violations get blocked, you can:

## PART 2) Sanitize and wrap inputs in `TrustedHTML`s, in one place

Policy objects can be created to generate `TrustedHTML` to tell the browser that the input is safe.

```js
if (window.trustedTypes?.createPolicy) {
  // create a policy object to call later to sanitize a string:
  const escapeHTMLPolicy = trustedTypes.createPolicy("myEscapePolicy", {
    createHTML: (value, type, source) => {
      console.log('got into myEscapePolicy createHTML', value, type, source);
      if (source === 'eval') {
        // do something
      }
      return value.replace(/\</g, '&lt;');
      // this sanitized string will get wrapped in a TrustedHTML
    },
    createScript: (value, type, source) => value, // gets wrapped in a TrustedHTML
    createScriptURL: (value, type, source) => value, // gets wrapped in a TrustedHTML
  });

  // call our escapeHTMLPolicy object to sanitize a string and wrap it in a TrustedHTML:
  const escaped = escapeHTMLPolicy.createHTML('<img src=x onerror=alert(1)>');
  el.innerHTML = escaped; // '&lt;img src=x onerror=alert(1)>'
  console.log(escaped instanceof TrustedHTML); // true
  
  // or a default policy: (use sparingly!)
  trustedTypes.createPolicy('default', {
    createHTML: (value, type, source) => {
      // use a library to sanitize and then this policy will wrap it in a TrustedHTML:
      // DOMPurify.sanitize(value, {RETURN_TRUSTED_TYPE: true})
      console.log('got into default policy createHTML', value, type, source);
      return value;
    }
  });
}
```

## PART 3) Set up an event to trigger for and report security violations

Since `trustedTypes` is turned on, you can listen for `securitypolicyviolation`s:

```js
document.addEventListener('securitypolicyviolation', (e) => {
  console.log('\n\nDetected securitypolicyviolation:');
  console.error(e);
  alert('Detected securitypolicyviolation. \n\nSee browser console log for details on this object: \n\n' + e);
});
```

```html
<head>
  <!-- you can make report violations to some site //my-csp-endpoint.example with: -->
  <meta http-equiv="Content-Security-Policy-Report-Only" content="require-trusted-types-for 'script'; report-uri //my-csp-endpoint.example">
  <!-- or: -->
  <meta http-equiv="Content-Security-Policy" content="require-trusted-types-for 'script'; report-uri //my-csp-endpoint.example">
</head>
```

## Further resources

https://www.youtube.com/watch?v=IeKLIwJ2ZMY

https://web.dev/articles/trusted-types

check browser support (JS): https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API#examples

check browser support (CSP): https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/trusted-types#syntax
