# Why test?

It's important to be sure our app works as expected, especially when adding new features or changing existing ones.

Our tests can even be automated.

Because testing takes time and money, we should have a testing priority:

1. High value features - Focusing on the 'happy paths (what we want the user to do)'
2. Edge cases in high value features - Focusing on the 'sad paths (following something weird)'
3. Things that are easy to break
4. Basic react component testing - Focusing on components used _very_ often in the codebase
   1. User interactions
   2. Conitional rendering
   3. Utils/Hooks

We shouldn't test implementation, e.g. opening the devtools and making sure our state changed, rather we should focus our tests in the way in which the user will use our app.

## Three most common tests

### Unit testing

Testing a very small part of the code, e.g. a function.

### Integration testing

Finding out if multiple units of the app work correctly together, pretty much combining multiple unit tests into one larger test.

### End-to-end testing

Testing from frontend to backend. Mimicking how the user would really use our application in the browser.

Note that once there are no errors, making no assertions in a test means it will automatically pass.
We can use this log to find out what 'names' are available in our screen/component.

#### Log the HTML that was found

```js
test("On render, the button is disabled", () => {
  render(<MyButton />);

  screen.debug();
});
```

#### We can use this to see what roles are available in an element. E.g. the result will group buttons together

```js
test("On render, the button is disabled", () => {
  render(<MyButton />);

  screen.getByRole("");
});
```

#### Target a specific role and name (a button with a name of 'pay')

```js
test("On render, the button is disabled", () => {
  render(<MyButton />);

  // case insensitive search
  expect(screen.getByRole("button", { name: /pay/i })).toBeDisabled();
});
```

#### Wait for something to appear onscreen

```js
test("On render, the button is disabled", async () => {
  render(<MyButton />);

  expect(await screen.findByRole("button", { name: /pay/i })).toBeDisabled();
});
```

#### User events

```js
test("Button is enabled when amount input has text", () => {
  render(<MyForm />);

  // Receives the string to be typed
  userEvent.type(screen.getByPlaceholderText(/amount/i), "50");

  // Then can make some assertion with our text entered
  expect(await screen.findByRole("button", { name: /pay/i })).toBeEnabled();
});
```

## Arrange, Act, Assert pattern

We follow this pattern:

1. Rendering a component

2. User events, typing, clicking

3. Making our assertions

# Integration tests

It's important to think about realistic user flows. The examples above aren't really how a user would use the application. E.g. a user might login, add some data to the form, then click on the submit button. Those things don't really happen in isolation.

So we could combine all of those individual unit tests into a single integration test.

It's not just about combining unit tests into one, but if you find you can combine them and create a more realistic user flow, you should do it. Also sometimes it's more practical to write one big integration test than many little unit tests.

# End-to-end testing with Cypress

We'd add `yarn add -D cypress @testing-library/cypress`

To start it `yarn run cypress open`

To add react-testing-library cypress commands so we can use the same commands as above, but with cypress:

Go to `cypress/support/commands.js`

Replace contents with:

`import "@testing-library/cypress/add-commands"`
