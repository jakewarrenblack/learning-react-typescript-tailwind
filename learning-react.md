# React Recap and Learning

## 1. Thing number one I knew how to do but didn't understand

```js
const Dog = ({ dogInfo }) => {
  return (
    <div>
      <h1>{name}</h1>
      <h1>{age}</h1>
      <h1>{breed}</h1>
    </div>
  );
};

const DogPage = () => {
  return (
    <div>
      <Dog
        dogInfo={{
          name: "Lola",
          breed: "Jug",
          age: 4,
        }}
      />
    </div>
  );
};

// Curly brackets in Dog's props are object destructuring, it would be the same thing to do it this way:

const Dog = (props) => {
    return (
        <div>
            <h1>{props.dogInfo.name}</h1>
            // etc
        </div>
    )


const DogPage = () => {
    return (
        <div>
        <Dog dogInfo={{
            name: 'Lola'
        }}>
        </div>
    )
}}
```

---

## 2. Using the spread operator to forward props

Something like the above is a bit repetitive, when we're passing everything that's in an object anyway, we can pass everything at the same time with the spread operator.

```js
const Dog = ({ name, age, breed }) => {
  return (
    <div>
      <h1>{name}</h1>
      <h1>{age}</h1>
      <h1>{breed}</h1>
    </div>
  );
};

const DogPage = (props) => {
    return (
        <div>
        // If we had the data we wanted in props, we could just pass everything together like this
        <Dog {...props}>
        </div>
    )
}
```

## 3. Passing JSX as children

When we nest content within a JSX tag, the parent component gets that JSX inside a prop named _children_, which we can wrap in anything we want.

```js
const Dog = ({ children }) => {
  return (
    <div style={{ backgroundColor: "red" }}>
        {children}
    </div>;
  )
};

export default DogPage = () => {
    return (
        <Dog>
            <h1>Whatever content, doesn't matter</h1>
        </Dog>
    )
}

// Dog doesn't need to know what's being rendered within it, just wraps around any nested content
```

So a component with a 'children' prop is like a component with a hole in it, you can plug anything into it.

---

## 4. Logical AND operator

This operator is like a ternary with only one side.

```js
const Dog = () => {
  return <div>{isBrown && "Yep!"}</div>;
};
```

So here, we'll render 'Yep!' if brown is true, and nothing if brown is false.

---

## 5. You can assign JSX to a variable

```js
const Dog = ({ isBrown }) => {
  let myJSX = (
    <div>
      <h1>Not brown</h1>
    </div>
  );

  if (isBrown) {
    myJSX = (
      <div>
        <h1>Definitely brown</h1>
      </div>
    );
  }

  return <div>{myJSX}</div>;
};
```

---

## 6. State as a snapshot / Rendering

### Rendering

Components render for one of two possible reasons:

1. It's the component's first render
2. The component's state has been updated

That's why we see a react app calling this:

```js
ReactDOM.render(
  <App/>
  document.getElementById('root')
)
```

It's just triggering the initial render.

---

We can then trigger **further** renders by setting the state.

Updating our component's state automatically queues a new render.

Keeping in mind that components are functions, on the initial render, React calls the root component. On subsequent renders, React calls the function component whose state update triggered a new render.

On initial render, React creates all the necessary DOM nodes.

On a re-render, React finds out what changed in the DOM since the last render.

---

Finally, it commits changes to the DOM. First render uses appendChild() to put everything on the screen. On re-renders, React does the minimum necessary amount of work to make the DOM match the output of the latest render.

## When we call 'setState'

Setting state doesn't change the variable we already have, but queues a new render. So maybe an event handler runs, and sets the state.

1. Event handler executes
2. State is set, a re-render is triggered
3. The UI is re-rendered to correspond with the new state value

**Keep in mind that setting state only tells React it has to be changed on the _next_ render! Each render's state values are fixed.**

By the above, we mean:

```js
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

Calling the above won't make 'number' equal 3. It will just make it 1, 3 times.

It says to itself 'what's the value of number? It's zero. Okay, get ready to increase that by one on the next render. And then it does the same thing twice again, because the state value for that render hasn't changed, it _won't_ change until the next render.

## 7. Do something with the state value, instead of just replacing it

If we want to update the same state value a few times before re-rendering, we can pass a _function_ to the setState, to calculate the next state based on the previous one in the queue.

```js
setNumber((old) => old + 1);
setNumber((old) => old + 1);
setNumber((old) => old + 1);

// Number will then be 3
```

That's called an updater function.

## 8. Updating an object in state

If we have this:

```js
const [number, setNumber] = useState(0);

setNumber(5);
```

We haven't actually _changed_ the number 0, how could we? We just replaced it with a different number.

Primitives like numbers, booleans, and strings are immutable, they can't be changed. We can just replace them.

Objects and arrays, however, are mutable.

We could do this type of thing:

```js
const [dog, setDog] = useState({
  name: "Ted",
  age: 5,
  breed: "Golden retriever",
});

dog.name = "Billy";
```

Or this:

```js
const [dogs, setDogs] = useState([
  {
    name: "Ted",
    age: 5,
    breed: "Golden retriever",
  },
  {
    name: "Billy",
    age: 10,
    breed: "Labrador",
  },
]);

dogs[0].name = "Tom";
```

Except we shouldn't do either of those things.

It's better to treat any javascript object or array as read-only.

In either case, we don't mutate values directly, we make new values and replace the existing ones.

---

## For an object, we can do this:

```js
const [dog, setDog] = useState({
  name: "Tom",
  age: 5,
  breed: "Jack Russell",
});

setDog({
  ...dog,
  name: "Billy",
});
```

We can use the spread operator to copy everything, then we override the thing we wan't to change.

So there we just made a copy of the existing dog, replaced the one value we wanted to, and sent our new object to the state, no mutation.

## For an array:

Adding to an array:

```js
const [dogs, setDogs] = useState([
  {
    name: "Tom",
    age: 5,
    breed: "German Shepherd",
  },
  {
    name: "Rodney",
    age: 14,
    breed: "Chihuahua",
  },
]);

setDogs([...dogs, { name: "Paddy", age: 2, breed: "Husky" }]);
```

Removing from an array:

```js
const [dogs, setDogs] = useState([
  {
    name: "Tom",
    age: 5,
    breed: "German Shepherd",
  },
  {
    name: "Rodney",
    age: 14,
    breed: "Chihuahua",
  },
]);

// Make a new array with just Tom in it, filter out anybody not named 'Tom'
setDogs(dogs.filter((dog) => dog.name === "Tom"));
```

Modify an item in the array:

```js
const [dogs, setDogs] = useState([
  {
    name: 'Tom',
    age: 5,
    breed: 'German Shepherd'
  },
  {
    name: 'Rodney',
    age: 14,
    breed: 'Chihuahua'
  }
])

// We're just making a new array
// If name is not Tom, add it, unchanged, to our new array
// If name is Tom, change his age and add it
setDogs(dogs.map((dog) => {
  if(dog.name === 'Tom'){
    return {
      ...dog,
      age: 8
    }
    else{
      return dog
    }
  }
}))
```

Above changes Tom's age to 8.

---

Replacing an item at some index:

```js
const changeDog = (index) => {
  setDogs(
    dogs.map((dog, i) => {
      if (i === index) {
        return {
          name: "Joe",
          age: 10,
        };
      } else {
        return dog;
      }
    })
  );
};

// Will replace dog at index 1 with Joe, aged 10
changeDog(1);
```

## 9. useRef

Refs are a bit like useState except they don't trigger re-renders. They do persist across renders.

An example is if we want to have a counter showing how many times a component has rendered. We could try to do it with useState, but that wouldn't work.

```js
import react, {useState, useEffect} from 'react';

const [renderCount, setRenderCount] = useState(0);
const [text, setText] = useState('');

useEffect(() => {
  setRenderCount((prevRenderCount) => prevRenderCount + 1)
})

const App = () => {
  return (
    <div>
      <input
        type="text"
        value={input}
        onChange={(e) => setText(e.target.value)}
      />
      <br/>

      <p>You entered {text}<p>
      <p>I rendered {renderCount} times</p>
    </div>
  )
}
```

That wouldn't work, because useEffect would be stuck in an infinite loop. Changing the state triggers a re-render, so it'd just keep going.

If we used a ref instead of state for the render count, it would work perfectly.

We can also use refs to get a reference to a DOM element. Common use case is focusing buttons or triggering hover states.

```js
import react, { useState, useEffect } from "react";

const myRef = useRef();
const [text, setText] = useState("");

const focusButton = () => {
  myRef.current.focus();
};

const App = () => {
  return (
    <div>
      <input
        ref={myRef}
        type="text"
        value={input}
        onChange={(e) => setText(e.target.value)}
      />
      <br />

      <button onClick={focusButton}>Focus the input</button>
    </div>
  );
};
```

The above will focus the input element when we click the button. Every html element in react can have a ref like our input does above.

## When we don't have access to the ref directly

If we have some custom component with an input inside it, for example, we won't have direct access to the ref it holds. This is where **forwardRef** comes in.

```js
import react, { useState, useEffect } from "react";

const myRef = useRef();
const [text, setText] = useState("");

const focusButton = () => {
  myRef.current.focus();
};

// Wrap the component that needs the forwarded ref in the React.forwardRef function

// And notice how we pass two properties now, the props, *and* the ref
const MyCustomInput = React.forwardRef((props, ref) => {
  // Spread operator to use all props at once
  return <input ref={ref} {...props}/>
})

const App = () => {
  return (
    <div>
      <!-- Now we can use our ref as normal, it goes straight into the real input element, rather than our custom component itself -->
      <MyCustomInput
        ref={myRef}
        type="text"
        value={input}
        onChange={(e) => setText(e.target.value)}
      />
      <br />

      <button onClick={focusButton}>Focus the input</button>
    </div>
  );
};
```

## 10. useReducer

Good for when state management is getting a bit unwieldy.

Can go for useState or useReducer, sort of does the same thing.

Maybe if we had a load of state changing methods like in a todo app:

```js
// User presses 'Add'
const addDog = (dog) => {
  setDogs([
    ...dogs,
    {
      id: 1
      name: 'Billy',
      age: 5,
      breed: 'Pug'
    }
  ])
}

// User changes something, presses 'save'
const updateDog = (dog) => {
  setDogs(dogs.map((d) => {
    if(d.id === dog.id){
      return dog
    }
    else{
      return d;
    }
  }))
}

// User presses 'delete'
const removeDog = (dogId) => {
  setDogs(dogs.filter((d) => d.id !== dogId))
}
```

Instead of setting state directly like this, we dispatch actions. This reminds me of Vuex.

We can write a reducer function and use the reducer function from our component then.

We change our thinking from 'here's what to do', to 'the user just did this'. Which is why we use dispatch actions.

Write like this:

```js
// User presses 'Add'
const addDog = (dog) => {
  dispatch({
    type: "added",
    dog: dog,
  });
};

// User changes something, presses 'save'
const updateDog = (dog) => {
  dispatch({
    type: "changed",
    dog: dog,
  });
};

// User presses 'delete'
const removeDog = (dogId) => {
  dispatch(
    // An action object:
    {
      type: "deleted",
      id: dogId,
    }
  );
};
```

The object we passed to 'dispatch' above is an 'action'.
