# Learning TypeScript

## 1. Creating a type

`type Person = { name: string, height: number, isLeftHanded: boolean, }`

### Using that type:

```ts
const myPerson: Person = {
  name: "Joe Bloggs",
  height: "160",
  isLeftHanded: true,
};
```

### A function returning a specific type:

_(Keep in mind it's bad for a method to change the original person object, instead should return a **new** object)_

```ts
function editName(person: Person): Person{
	return{
		person.name = 'Billy Bob'
		return person;
	}
}
```

---

## 2. ReadOnly properties

### Using TypeScript to prevent the above mistake:

```ts
type Person = {
  readonly name: string;
  readonly height: number;
  readonly isLeftHanded: boolean;
};
```

So instead ts would force us to do it like this:

```ts
function editName(person: Person) {
  return {
    name: "Billy Bob",
    height: "180",
    isLeftHanded: false,
  };
  // Returns a *new* person instead, which is fine
}
```

There's also the readonly 'mapped' type, which is like this:

```ts
type Person = Readonly<{
  name: string;
  height: number;
  isLeftHanded: boolean;
}>;
```

This is just handier than writing 'readonly' for every attribute.

_Or_ we can cast one type into another, like this:

```ts
type Dog = {
  isBrown: boolean;
};

type ReadonlyDog = Readonly<Dog>;
```

That'll make the property 'isBrown' readonly.

That's called a **'Mapped'** type. There are others like Required and Partial.

---

## 3. Array Types, Literal Types, Intersection types

```ts
function makeAllLeftHanded(people: Person[]): Person[] {
  // Has to receive and return an array of people
  const newPeople: Person[] = people.map((person: Person) => {
    return { ...person, isLeftHanded: true };
  });
  return newPeople;
}
```

Could make sure the original array isn't modified here using readonly. We don't use the Readonly<> mapped type for arrays.

```ts
function makeAllLeftHanded(people: readonly Person[]): Person[] {
  // Means ts will stop us from doing something like:
  person[0] = { name: "Tom Cruise", height: "120", isLeftHanded: true };

  // Or

  people.push({ name: "Hank Hill", height: "200", isLeftHanded: true });

  // Because either would be modifying the original array
}
```

### Literal types

```ts
type LeftHandedPerson = Readonly<{
  name: string;
  height: number;
  isLeftHanded: true;
}>;
```

We can use an exact value when specifying types too, which is a **literal type**.

```ts
const myLefty: LeftHandedPerson = {
  name: "Ned Flanders",
  height: "160",
  isLeftHanded: false,
};

// This won't compile!
// Will tell us 'type 'false' is not assignable to type 'true'.
```

So it would then make sense to use the LeftHandedPerson type for our makeAllLeftHanded method.

```ts
function makeAllLeftHanded(people: Person[]): LeftHandedPerson[] {
  const leftHandedPeople: LeftHandedPerson[] = people.map((person: Person) => {
    return { ...person, isLeftHanded: true };
  });
  return leftHandedPeople;
}
```

---

## 4. Intersection types

We do have duplicate code between the Person and LeftHandedPerson types.

We can have intersection types which have all the properties of the types given to it:

```ts
type Cat = { hasStripes: boolean };
type Zebra = { numberOfStripes: number };

type CatandZebra = Cat & Zebra;

// Above is the same as:

type CatandZebra = {
  hasStripes: boolean;
  numberOfStripes: number;
};
```

**Also**, if the second type is more specific than the first, the second type overrides the first. Like this:

```ts
type Cat = { hasStripes: boolean };
type Zebra = { hasStripes: true };

// Both have the property 'hasStripes', but zebra's is more specific. True is more specific than boolean.

type CatandZebra = Cat & Zebra;

// Above is the same as:

type CatandZebra = { hasStripes: true };
```

We can apply the same idea to our left handed person.

```ts
type Person = Readonly<{
  name: string;
  height: number;
  isLeftHanded: boolean;
}>;

// Overrides the isLeftHanded property
type LeftHandedPerson = Person & { readonly isLeftHanded: true };
```

**Summing this up**, we could write our makeAllLeftHanded method like this:

```ts
const people: Person[] = [
    {
        name: 'Billy bob',
        height: 160,
        isLeftHanded: false,
    },
    {
        name: 'Joe Bloggs',
        height: 140,
        isLeftHanded: true,
    }
]

function makeAllLeftHanded(people: Person[]): LeftHandedPerson[]{
    return people.map((person) => {
        ...person,
        isLeftHanded: true,
    })
}
```

---

## 3. Union types and optional properties

If we wanted to have several valid options for a type, we could use a union type.

For example:

```ts
type Person = Readonly<{
  name: string;
  height: number;
  language: "english" | "polish" | { other: string };
}>;

// So then any of the below are valid:

const person1: Person = {
  name: "Joe Bloggs",
  height: 160,
  language: "english",
};

const person2: Person = {
  name: "Billy Bob",
  height: 140,
  language: "polish",
};

const person3: Person = {
  name: "Tom Cruise",
  height: 120,
  language: { other: "chinese" },
};
```

We could make language its own type:

```ts
type Language = "english" | "polish" | { other: string };

// And then assign like this:

type Person = {
  name: string;
  height: number;
  language: Language;
};
```

## 5. Optional properties

We can append a question mark to a type's name to make it optional:

```ts
type Person = Readonly<{
  name: string;
  height: number;
  language: Language;
  nationality?: string;
}>;

// So then this is valid:

const myPerson: Person = {
  name: "Billy Bob",
  height: 180,
  language: "english",
};

// We don't need to provide nationality, if we don't want to
```

# Generics

Prerequisite for my pea brain to understand the below examples.

## JS Closure

```js
function outer() {
  var a = 10;

  function inner() {
    var b = 5;
    console.log(`a= ${a}, b= ${b}`);
  }
  return inner;
}

var x = outer();
```

What happens when we run x:

- JS reads the method outer, creates the variable a
- Sees inner(), it's a function declaration, don't do anything
- Sees the return for inner, reads the body of inner and returns it
- Outer() has finished execution, all the variables within its scope cease to exist

The variable _x_ now contains the body of the function '_inner_'.

What if we were to then do this:

```js
var y = x();
```

We'd be executing the function '_inner_'.

What then happens

- Variable _b_ is created and set to 5
- JS tries to run console.log on _a_ and _b_
- But _a_ no longer exists! It ceased to exist when outer() finished running
- _Except_ inner() can access the variables of its surrounding function.
- inner() preserved the scope chain of outer() when outer() was executed,
- so it can access outer()'s variables!
- when we ran outer() the first time, inner() was stored in y, and held onto the value of _a_!

Explanation of this could be wrong, but I think this is how closure works in JS.

Felt that was a good precursor to following https://ts.chibicode.com/generics

---
