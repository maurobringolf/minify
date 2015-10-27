# babel-minify

```sh
$ git clone https://github.com/facebook/babel-minify
$ npm install
$ node index.js fb.js
```


current numbers:
```
babel    232.97kB 82%     51.58kB 57%      17ms
uglify   213.49kB 98%     54.02kB 50%      16ms
closure  209.5kB  102%    54.45kB 49%      20ms
fb jsmin 235.99kB 80%     57.84kB 40%      18ms
```

After refactor and bug fixing:
```
uglify   213.49kB 98%     54.02kB 50%      15ms
closure  209.5kB  102%    54.45kB 49%      21ms
babel    229.58kB 85%     56.61kB 43%      18ms
fb jsmin 235.99kB 80%     57.84kB 40%      18ms
```

Major bug was that a variable name might've been used everywhere which is amazing for gzip.

After a better job with the variable encoding and fixing a bug not renaming the params:
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      18ms
closure 209.5kB  102%    54.45kB 49%      17ms
babel   216.96kB 95%     54.92kB 47%      18ms
jsxmin  235.99kB 80%     57.84kB 40%      17ms
```

After fixing a DCE bug that randomly removed functions:
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      0ms
closure 209.5kB  102%    54.45kB 49%      0ms
babel   219.49kB 93%     55.64kB 46%      0ms
jsxmin  235.99kB 80%     57.84kB 40%      0ms
```

After adding better return statement merging:
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      0ms
closure 209.5kB  102%    54.45kB 49%      0ms
babel   218.71kB 94%     55.5kB  46%      0ms
jsxmin  235.99kB 80%     57.84kB 40%      0ms
```

```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      0ms
closure 209.5kB  102%    54.45kB 49%      0ms
babel   218.59kB 94%     55.49kB 46%      0ms
jsxmin  235.99kB 80%     57.84kB 40%      0ms
```

moar return merging:
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      0ms
closure 209.5kB  102%    54.45kB 49%      0ms
babel   216.18kB 96%     54.83kB 48%      0ms
jsxmin  235.99kB 80%     57.84kB 40%      0ms
```

for merging:
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      0ms
babel   214.5kB  98%     54.32kB 49%      0ms
closure 209.5kB  102%    54.45kB 49%      0ms
jsxmin  235.99kB 80%     57.84kB 40%      0ms
```

Fixing a bug in not pushing var decl when merging into seq expr :(
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      14ms
closure 209.5kB  102%    54.45kB 49%      21ms
babel   218.78kB 94%     55.57kB 46%      18ms
jsxmin  235.99kB 80%     57.84kB 40%      15ms
```


mangling enhancement:
```
        raw      raw win gzip    gzip win parse time
uglify  213.49kB 98%     54.02kB 50%      15ms
babel   218.66kB 94%     54.06kB 50%      17ms
closure 209.5kB  102%    54.45kB 49%      17ms
jsxmin  235.99kB 80%     57.84kB 40%      14ms
```

TODOS:


- convert early if returns to if with negated condition (babel)
- needlessly parenthesising expressions (babel)
- understand and optimize continue
- some conditionals (with void 0) maybe turned into logical exprs
- should merge more blocks into sequence expressions

- Here is a place were uglify is crushing it making it into two fors and one if with no blocks:

```
  // taken from require.js
 function _debugUnresolvedDependencies(names) {
    var unresolved = Array.prototype.slice.call(names);
    var visited = {};
    var ii, name, module, dependency;

    while (unresolved.length) {
      name = unresolved.shift();
      if (visited[name]) {
        continue;
      }
      visited[name] = true;

      module = modulesMap[name];
      if (!module || !module.waiting) {
        continue;
      }

      for (ii = 0; ii < module.dependencies.length; ii++) {
        dependency = module.dependencies[ii];
        if (!modulesMap[dependency] || modulesMap[dependency].waiting) {
          unresolved.push(dependency);
        }
      }
    }
}
```

- multiple conditionals should add an "&&" between them
- why is this even real code? :/
```
var symbolIterator=typeof Symbol === 'function'?typeof Symbol === 'function'?
    Symbol.iterator:'@@iterator':
    '@@iterator'
```

should compile to

```
l = "function" == typeof Symbol && "function" == typeof Symbol ? Symbol.iterator : "@@iterator",
```