Writing your own plugin

A tailwind config is like this:

```js
module.exports = {
  theme: {
    extend: {},
  },
  variants: {},
  plugins: [],
};
```

The plugin:

**In a file called 'named-colors.js'**

```js
const plugin = require("tailwindcss/plugin");

// allow the user to pass options
// it receives an anonymous function
module.exports = plugin.withOptions(() => {
  return function ({ addUtilities }) {
    // we can pick the utilities we want to use by passing them into parameters
    addUtilities({
      ".bg-coral": { background: "coral" },
    });
  };
});
```

At this point, we've defined the plugin but haven't used it. We have to add it to our plugins:

```js
module.exports = {
  theme: {
    extend: {},
  },
  variants: {},
  plugins: ["./named-colors"],
};
```

We can provide an object as the second argument to the 'plugin' function, which will allow us to merge our own config values into the tailwind.config:

```js
module.exports = plugin(
  function ({ matchUtilities, theme }) {
    matchUtilities(
      {
        tab: (value) => ({
          tabSize: value,
        }),
      },
      { values: theme("tabSize") }
    );
  },
  {
    theme: {
      tabSize: {
        1: "1",
        2: "2",
        4: "4",
        8: "8",
      },
    },
  }
);
```
