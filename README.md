# Learning `trustedTypes`

Just one of the things I'm learning. https://github.com/hchiam/learning

Prevent DOM XSS. Page-wide/centrally, instead of trying to manually find and fix all XSS sinks.

Demo: https://codepen.io/hchiam/pen/mdYOZOP?editors=0010

Requires browser support and [CSP (Content Security Policy)](https://github.com/hchiam/learning-csp) to turn on `trustedTypes`:

```html
<head>
  <meta http-equiv="Content-Security-Policy" content="require-trusted-types-for 'script'">
</head>
```

Then in JS:

```js
document.addEventListener('securitypolicyviolation', (e) => {
  console.log('\n\nDetected securitypolicyviolation:');
  console.error(e);
  alert('Detected securitypolicyviolation. \n\nSee browser console log for details on this object: \n\n' + e);
});
```

```js
try {
  element.innerHTML = '<img src=x onerror=alert("xss")>';
} catch(e) {
  console.log('\n\nxss:');
  console.error(e);
  alert('Blocked DOM XSS: \n\n' + e);
}
```

## There's also:

- detection of browser support for trustedTypes.
- creation of page-wide trustedTypes policy, instead of trying to find and fix each XSS sink.
- sanitization using trustedTypes.

## Further resources

https://www.youtube.com/watch?v=IeKLIwJ2ZMY

https://web.dev/articles/trusted-types

check browser support (JS): https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API#examples

check browser support (CSP): https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/trusted-types#syntax
