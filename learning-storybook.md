# Learning Storybook

## 1. Creating a basic story

We create a file named something like 'MyComponent.stories.js'.

We set up our story inside:

```js
import MyButton from '../components/MyButton'

export default {
    title: 'MyComponent'
    component: MyComponent
}

// The name we give this will be the name of our story, so we just return our component
export const MyAmazingComponent = () => <MyComponent someProp="Hello world" />
```

If we have propTypes or TypeScript, storybook will be clever enough to look at our prop types and create little docs for the component based on them.

## 2. Add interactivity

The template is a kind of base function:

```js
// Arguments passed to Template get passed into button
// So Template returns the button with all of its props
const Template = (args) => <Button {...args} />;

// If we want to copy the Template function and reuse it
export const SomeComponent = Template.bind({});

SomeComponent.args = {
  backgroundColor: "red",
  label: "Hello world",
  size: "md",
};
```

So the above gives us controls we can actually modify.

## 3. Defining an action

```js
export default {
    title: 'MyComponent',
    component: MyComponent,
    // Just giving it the prop name and an action with a label
    // 'action' is telling storybook that handleClick **does something**
    argTypes: {handleClick {action: "handleClick"} }
}
```

If we used something like onClick as a name, storybook would be clever enough to realise that's an event handler.

## 4. More complicated stories
