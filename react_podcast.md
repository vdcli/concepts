# 🎙️ Component Thinking
## The React Podcast — Building UIs from Absolute Zero

> **Episode Runtime:** ~210 Minutes | **Format:** Conversational Two-Host
>
> **SAM** — The learner: a backend developer who's never touched web/UI code, asks the exact questions a newcomer would ask.
> **JORDAN** — The React expert: patient, analogy-obsessed, never assumes prior knowledge, always explains the "why" before the "how."

---

## 📋 Speaker Guide

- **[SAM]** — The newcomer voice. Asks the questions anyone new to web and UI would ask.
- **[JORDAN]** — The expert voice. Builds understanding layer by layer, never skips steps.
- `🎵` — Sound/music direction for the audio producer.
- `📝` — Production notes.

---

## Topics at a Glance

| Time | Topic | Key Concepts |
|------|-------|--------------|
| 0:00–12:00 | What even is a UI? The web from zero | Browser, HTML, DOM, why UIs are hard |
| 12:00–22:00 | What is React and why does it exist? | Declarative vs imperative, virtual DOM |
| 22:00–34:00 | Components — the core idea | Functions as UI, JSX, composition |
| 34:00–46:00 | Props — passing data in | One-way data flow, immutability, prop types |
| 46:00–62:00 | State — making things change | useState, re-renders, state vs props |
| 62:00–76:00 | Handling events | onClick, onChange, synthetic events, pitfalls |
| 76:00–88:00 | Conditional rendering | &&, ternary, early return patterns |
| 88:00–100:00 | Lists and Keys | .map(), key prop, why keys matter |
| 100:00–116:00 | useEffect — talking to the outside world | Side effects, dependencies, cleanup |
| 116:00–126:00 | useRef — remembering without re-rendering | DOM refs, mutable values, pitfalls |
| 126:00–140:00 | useMemo & useCallback — performance tools | When to optimize, when NOT to |
| 140:00–152:00 | useContext — sharing data without prop drilling | Context API, provider pattern, pitfalls |
| 152:00–164:00 | Custom Hooks — your own reusable logic | Extracting logic, naming rules, patterns |
| 164:00–176:00 | Component composition patterns | Children prop, slots, compound components |
| 176:00–188:00 | Forms and controlled inputs | Controlled vs uncontrolled, validation |
| 188:00–198:00 | React Router — navigation basics | Routes, links, URL params, nested routes |
| 198:00–210:00 | Best practices, pitfalls & mental models | Top 10 mistakes, production patterns |

---

# SEGMENT 1: What Even Is a UI? The Web from Zero `(0:00 – 12:00)`

🎵 *Warm, approachable intro music plays for 5 seconds then fades*

**[SAM]:** Welcome to Component Thinking — the podcast that assumes you know nothing about web development and goes from there. I'm Sam. I'm a backend Java developer and I've spent my whole career writing APIs, services, databases. I have genuinely never written a single line of HTML in a professional context.

**[JORDAN]:** And I'm Jordan. I've been building React UIs for about seven years now. And Sam, I want to start today completely from scratch. Not "here's a React component" scratch. I mean: why does a browser exist, what problem is it solving, and what even is a user interface. Because if we don't understand that, nothing about React will make intuitive sense.

**[SAM]:** I appreciate that. Because honestly when someone says "it's just JavaScript," I feel like I'm missing about fifty steps before that makes sense.

**[JORDAN]:** You're missing a lot of steps and nobody talks about them. Let's fix that.

**[SAM]:** Okay so — what is a browser? Like obviously I use Chrome every day but what is it actually doing?

**[JORDAN]:** A browser is a program that fetches files from the internet and renders them visually. That's it at its core. You type a URL, the browser makes an HTTP request — which you know from backend work — gets back a file, and then it does something you've never had to think about: it draws pixels on screen based on the contents of that file.

**[SAM]:** And the file it gets back is HTML?

**[JORDAN]:** Almost always, yes. HTML — HyperText Markup Language — is a text format for describing the *structure* of a page. Think of it like a really verbose JSON that describes a tree of elements. You have a button here, a paragraph of text there, an image, a list. HTML describes what things *are*.

**[SAM]:** Not how they look?

**[JORDAN]:** Not how they look. That's CSS — Cascading Style Sheets. CSS says: "that button should be blue, 48 pixels tall, with rounded corners." HTML is structure, CSS is appearance. And then there's a third piece: JavaScript. JavaScript is the programming language that runs *inside* the browser and can make the page *do things* — respond to clicks, fetch new data, change what's on screen without reloading the page.

**[SAM]:** Okay so it's three technologies working together.

**[JORDAN]:** Three technologies, always working together. And for most of the early web — we're talking 1995 to maybe 2010 — people wrote these three things pretty directly. You write your HTML, sprinkle some CSS, add a bit of JavaScript. Simple.

**[SAM]:** So what changed?

**[JORDAN]:** The *ambition* of web apps changed dramatically. Around 2008, 2010, companies started building things like Gmail, Google Maps, Facebook — apps that behave more like desktop software than traditional web pages. Gmail doesn't reload the page every time you open an email. Google Maps doesn't reload when you pan around. The page stays alive and updates *parts of itself* in response to what you do.

**[SAM]:** That sounds way harder to build.

**[JORDAN]:** Massively harder. And this introduces a core problem: the DOM.

**[SAM]:** The DOM. I've heard that word. What is it?

**[JORDAN]:** DOM stands for Document Object Model. When the browser reads your HTML file, it doesn't just display it as raw text. It parses it into a *tree of objects* in memory. Every element in your HTML becomes a node in this tree — a JavaScript object you can reach out and touch. You can say "find the button with this ID, change its text, add a CSS class to it." That tree of objects is the DOM.

**[SAM]:** Okay so the DOM is the browser's live, in-memory representation of the page.

**[JORDAN]:** Exactly. And the reason this matters is that manipulating the DOM directly — the old way — becomes a nightmare as your app grows. Imagine you're building a dashboard. A user filters a table. You have to: find the old rows in the DOM, remove them, create new row elements, insert them, update the count label, maybe change the color of a status badge. Each of those is a separate imperative command. And when you have hundreds of these interactions, the code becomes an absolute tangle of "find this element, change that attribute, update this other thing."

**[SAM]:** That sounds like writing JDBC code by hand to update ten different database tables for every operation.

**[JORDAN]:** That's a perfect analogy. And just like ORMs came along to give you a higher-level way to think about data, React came along to give you a higher-level way to think about UIs. That's the entire motivation.

---

# SEGMENT 2: What Is React and Why Does It Exist? `(12:00 – 22:00)`

**[SAM]:** So React is the ORM of the DOM?

**[JORDAN]:** [laughs] I'm going to steal that. React is a JavaScript library — built by Facebook in 2013 — that solves exactly the problem we just described. Instead of imperatively manipulating the DOM, React lets you *declare* what you want the UI to look like, and React figures out how to make the DOM match that description.

**[SAM]:** Declare. That word gets thrown around. What does that actually mean in practice?

**[JORDAN]:** Okay, great question. Think about two ways to order food. Imperative: "First, go to table three. Pick up the menu. Open it to page four. Look at item number seven. Take a pen. Write down 'burger, medium, no onions'. Walk to the kitchen. Hand the paper to the cook." You're specifying every single step.

**[SAM]:** That's a lot.

**[JORDAN]:** Versus declarative: "I want a burger, medium, no onions." You state *what you want* and someone else figures out how to make it happen. That's what React does. Instead of telling the browser step-by-step how to update the DOM, you just tell React: "given this data, the UI should look like *this*." React handles all the DOM manipulation.

**[SAM]:** So I describe the end state and React makes it happen.

**[JORDAN]:** Exactly. And this leads to the most important mental model in React: **your UI is a function of your data.** Or more precisely: `UI = f(state)`. Given the same data, React will always render the same UI. There's no mystery, no hidden DOM state you have to track. You just think about what the UI looks like for any given snapshot of your data.

**[SAM]:** That's actually a familiar concept from functional programming. Pure functions.

**[JORDAN]:** Exactly — React components *are* pure functions in spirit. Same input, same output, no side effects. We'll come back to what "side effects" mean later, but that mental model is the foundation of everything.

**[SAM]:** Okay but you mentioned the "virtual DOM." What is that?

**[JORDAN]:** The virtual DOM is React's internal optimization. Instead of updating the real browser DOM every time something changes — which is slow — React maintains a lightweight JavaScript copy of the DOM in memory. When your data changes, React first updates the virtual DOM, then compares the new virtual DOM with the previous version — a process called "diffing" — and then makes only the *minimal* set of changes needed to the real DOM.

**[SAM]:** Like a git diff but for the UI.

**[JORDAN]:** Perfect analogy. Git doesn't rewrite the whole repository every time you commit — it figures out the delta and only stores the changes. React does the same with the DOM. This is what makes React fast enough to build complex interactive UIs without the page feeling sluggish.

**[SAM]:** So React is: declarative UI plus smart DOM updates under the hood.

**[JORDAN]:** That's it. And the mechanism that makes it declarative — the core building block — is what React calls a component. Let's talk about that.

---

# SEGMENT 3: Components — The Core Idea `(22:00 – 34:00)`

**[SAM]:** Alright, components. I've heard this word in backend contexts too — microservices, modules. Is it the same idea?

**[JORDAN]:** Very similar spirit. A React component is a self-contained, reusable piece of UI. Think of it like a custom HTML element that you define. The browser gives you `<button>`, `<input>`, `<table>`. React lets you create `<UserCard>`, `<PriceChart>`, `<NavigationMenu>` — your own elements with their own logic and appearance.

**[SAM]:** And each component is a function?

**[JORDAN]:** Modern React: yes. A component is just a JavaScript function that returns a description of what the UI should look like. Here's the simplest possible example:

```jsx
function Greeting() {
  return <h1>Hello, world!</h1>;
}
```

**[SAM]:** Wait, that's... HTML inside JavaScript? That's legal?

**[JORDAN]:** It's not actually HTML. It looks like HTML, but it's JSX — JavaScript XML. JSX is a syntax extension for JavaScript that lets you write UI structure in a familiar HTML-like syntax right inside your JavaScript code. It gets transformed — compiled — into regular JavaScript function calls before the browser ever sees it.

**[SAM]:** So `<h1>Hello, world!</h1>` becomes what, exactly?

**[JORDAN]:** It becomes `React.createElement('h1', null, 'Hello, world!')` — a function call that creates a virtual DOM node describing an h1 element. But you never write that by hand. You write the JSX and the toolchain handles the transformation.

**[SAM]:** Okay. So JSX is just syntactic sugar for creating virtual DOM nodes.

**[JORDAN]:** Exactly. And it's incredibly powerful because it lets you mix your HTML structure and your JavaScript logic in one place, in a natural way. For example:

```jsx
function Greeting() {
  const name = "Sam";
  return <h1>Hello, {name}!</h1>;
}
```

Those curly braces `{}` are the escape hatch from JSX back into JavaScript. Anything inside curly braces is a JavaScript expression. So you can put variables, function calls, math — anything that evaluates to a value.

**[SAM]:** That's actually elegant. It's like template strings but for UI structure.

**[JORDAN]:** Great way to think about it. Now — key rules about JSX that trip people up constantly.

**[SAM]:** Hit me.

**[JORDAN]:** First: a component must return a *single root element*. You can't return two sibling elements at the top level.

```jsx
// ❌ This breaks
function BadComponent() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}

// ✅ Wrap in a div — or a Fragment
function GoodComponent() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// ✅ Fragment: renders nothing extra in the DOM
function AlsoGood() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}
```

**[SAM]:** What's a Fragment?

**[JORDAN]:** A Fragment is React's way of grouping elements without adding an extra DOM node. The `<>...</>` syntax is shorthand for `<React.Fragment>`. Use it when you need a wrapper for JSX but don't want an extra `div` cluttering your DOM structure.

**[SAM]:** Got it. What's the second rule?

**[JORDAN]:** Second: JSX uses `className` instead of `class` for CSS classes, because `class` is a reserved word in JavaScript.

```jsx
// ❌ Wrong
<div class="container">

// ✅ Correct
<div className="container">
```

And third: HTML attributes in JSX are camelCase. `onclick` becomes `onClick`, `onchange` becomes `onChange`, `tabindex` becomes `tabIndex`.

**[SAM]:** Because it's actually JavaScript, not HTML.

**[JORDAN]:** Exactly. These are JavaScript objects under the hood, not strings. Now let's talk about the most powerful thing about components: you compose them like Lego.

**[SAM]:** Compose?

**[JORDAN]:** You can use components inside other components. A `<Dashboard>` component can contain a `<Chart>`, a `<StatsPanel>`, and a `<UserTable>`. Each of those can contain smaller components. You build complex UIs from simple, focused pieces.

```jsx
function UserCard() {
  return (
    <div className="card">
      <Avatar />
      <UserInfo />
      <ActionButtons />
    </div>
  );
}
```

**[SAM]:** And `<Avatar>`, `<UserInfo>`, `<ActionButtons>` are all separate components.

**[JORDAN]:** Each one focused on one thing. This is the component model, and it's what makes large React applications manageable. You break your UI into a tree of focused components, just like you break backend systems into focused services.

**📝 Best Practice:** One component, one responsibility. If a component is doing too many things, split it. A good rule of thumb: if you can't describe what a component does in one sentence without the word "and," it should probably be two components.

---

# SEGMENT 4: Props — Passing Data In `(34:00 – 46:00)`

**[SAM]:** So components are great. But they seem static right now. Every time I render `<Greeting>` I get "Hello, world!" How do I make it say different names?

**[JORDAN]:** That's exactly the right question, and the answer is **props**. Props — short for properties — are the way you pass data *into* a component from the outside. They're the parameters of your component function.

**[SAM]:** So like function arguments.

**[JORDAN]:** Exactly like function arguments. Watch:

```jsx
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Using it:
<Greeting name="Sam" />
<Greeting name="Jordan" />
```

In JSX, anything you write as an attribute on a component — like `name="Sam"` — gets bundled into an object called `props` and passed to your component function.

**[SAM]:** So JSX attributes become the props object.

**[JORDAN]:** Yes. And in practice, you almost always destructure the props for cleaner code:

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

Same thing, just destructuring the props object directly in the function signature.

**[SAM]:** What kinds of things can you pass as props?

**[JORDAN]:** Anything. Strings, numbers, booleans, objects, arrays, functions, even other React components. Here's the thing though — JSX attribute syntax works slightly differently depending on the type.

String: use quotes.
```jsx
<Button label="Click me" />
```

Anything else: use curly braces.
```jsx
<Button
  count={42}
  isDisabled={false}
  user={{ name: "Sam", role: "admin" }}
  onPress={handleClick}
/>
```

**[SAM]:** The curly braces are the "this is JavaScript" escape hatch again.

**[JORDAN]:** Exactly. Always. Anything that isn't a plain string needs curly braces.

**[SAM]:** Now, one thing I've heard is that props are "read-only." Why?

**[JORDAN]:** This is the most important rule about props and people break it constantly as beginners. **A component must never modify its own props.** Props flow in one direction: from parent to child. The parent is in charge of the data; the child just receives and displays it.

**[SAM]:** What if the child needs to change the data?

**[JORDAN]:** The child doesn't change the data itself. Instead, the parent passes *a function* as a prop. The child calls that function when something happens, and the parent's function updates the data. The parent is always in control.

```jsx
function ParentForm() {
  function handleNameChange(newName) {
    // parent handles the change
    console.log(newName);
  }

  return <NameInput onNameChange={handleNameChange} />;
}

function NameInput({ onNameChange }) {
  return (
    <input
      type="text"
      onChange={(e) => onNameChange(e.target.value)}
    />
  );
}
```

**[SAM]:** So the child raises an event by calling a function prop, and the parent decides what to do.

**[JORDAN]:** Perfect summary. This is called **one-way data flow** or unidirectional data flow. Data flows down via props, events flow up via callback functions. It's one of the most important concepts in React and it makes your app's behavior predictable — you always know where data comes from and who's responsible for changing it.

**📝 Pitfall:** Beginners often try to mutate props directly. `props.name = "newName"` — never do this. React won't catch it but your UI will behave unpredictably and you'll have very hard-to-debug issues. Treat props as if they were frozen.

**[SAM]:** What about default props? What if someone forgets to pass a prop?

**[JORDAN]:** You handle that with default values in your destructuring:

```jsx
function Greeting({ name = "Guest" }) {
  return <h1>Hello, {name}!</h1>;
}
```

If `name` isn't passed, it defaults to "Guest". Clean, simple, no special API needed.

**[SAM]:** And there's something called PropTypes I've seen mentioned?

**[JORDAN]:** PropTypes is a library for documenting and runtime-checking the types of your props. It's optional and mostly used in older codebases. Modern React projects typically use TypeScript instead — it catches type errors at compile time, not runtime.

```tsx
// With TypeScript
interface GreetingProps {
  name: string;
  age?: number; // optional
}

function Greeting({ name, age = 0 }: GreetingProps) {
  return <h1>Hello, {name}! You are {age} years old.</h1>;
}
```

**[SAM]:** That looks like a Java interface.

**[JORDAN]:** Very similar concept. TypeScript is a superset of JavaScript that adds static typing. Almost all serious React projects use it. We'll write our examples in JSX for simplicity today, but in production: use TypeScript.

---

# SEGMENT 5: State — Making Things Change `(46:00 – 62:00)`

**[SAM]:** Okay so props are data passed in from the outside. But what about data that *belongs to* a component — like, whether a dropdown is open, what text the user typed, how many times a button was clicked?

**[JORDAN]:** You just described **state**. State is data that a component owns and manages itself. It's the component's memory. And unlike regular JavaScript variables, when state changes, React knows to re-render the component — to call the function again and update the UI to reflect the new data.

**[SAM]:** Why can't I just use a regular variable?

**[JORDAN]:** Watch what happens if you try:

```jsx
// ❌ This does NOT work
function Counter() {
  let count = 0;

  function increment() {
    count = count + 1;
    console.log(count); // this changes, but the UI won't!
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

You click the button, `count` changes in memory, but the UI never updates. Why? Because React doesn't know anything changed. React only re-renders a component when its **state** or **props** change. A regular variable is invisible to React.

**[SAM]:** Ah. So I need to tell React "hey, this value changed, please re-render."

**[JORDAN]:** Exactly. And that's what `useState` does. It's a React **hook** — a special function that lets your component "hook into" React's internal machinery.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

**[SAM]:** Let me break that down. `useState(0)` — the zero is the initial value. And it returns... an array with two things?

**[JORDAN]:** Exactly. `useState` always returns an array with exactly two elements: the current value of the state, and a function to update that value. The `const [count, setCount]` is just JavaScript array destructuring — you're naming those two elements whatever you want. Convention is `value` and `setValue`.

**[SAM]:** And calling `setCount` is what tells React to re-render?

**[JORDAN]:** Yes. When you call the setter function with a new value, React schedules a re-render. On the next render, the component function runs again, `useState` returns the new value, and your JSX is recalculated with the updated data. The UI updates.

**[SAM]:** So every render is the function running again from top to bottom?

**[JORDAN]:** Exactly. Every render is a full re-execution of your component function. React calls your function, gets back the JSX description, diffs it against the previous render's output, and updates only the parts of the DOM that changed.

**[SAM]:** That means variables defined inside the function are recreated every render?

**[JORDAN]:** Yes! This is something that trips people up a lot. Every time the component renders, every `const`, every inline function, every object literal inside the function body is created fresh. State is the *exception* — React preserves state values between renders via its internal memory.

**[SAM]:** What about objects and arrays in state?

**[JORDAN]:** This is a critical pitfall. State updates must be **immutable**. You never mutate state directly — you create a new copy.

```jsx
const [user, setUser] = useState({ name: "Sam", age: 30 });

// ❌ WRONG — mutating state directly
function birthday() {
  user.age = user.age + 1; // React has no idea this changed
  setUser(user); // passing the same object reference — React may not re-render!
}

// ✅ CORRECT — create a new object
function birthday() {
  setUser({ ...user, age: user.age + 1 });
}
```

**[SAM]:** Because React compares by reference?

**[JORDAN]:** Exactly. When you call `setUser`, React checks: is this a different value than what's already in state? For objects, "different" means a different reference. If you mutate the object in place and pass the same reference back, React sees the same reference and may bail out of re-rendering — and even if it doesn't, you've broken React's ability to track history and do proper diffing.

**[SAM]:** Same rule for arrays?

**[JORDAN]:** Same rule. Never push, pop, splice, or sort an array in state. Instead, create a new array:

```jsx
const [items, setItems] = useState(["apple", "banana"]);

// ❌ Wrong
function addItem() {
  items.push("cherry");
  setItems(items);
}

// ✅ Correct — spread creates a new array
function addItem() {
  setItems([...items, "cherry"]);
}

// ✅ Removing an item
function removeItem(index) {
  setItems(items.filter((_, i) => i !== index));
}

// ✅ Updating an item
function updateItem(index, newValue) {
  setItems(items.map((item, i) => i === index ? newValue : item));
}
```

**[SAM]:** The functional update form. I've seen `setCount(prev => prev + 1)` — what's that about?

**[JORDAN]:** Great observation. When your new state depends on the previous state, you should use the **functional update** form. Instead of passing a value directly, pass a function that receives the current state and returns the new state:

```jsx
// ❌ Risky — uses stale closure
setCount(count + 1);

// ✅ Safe — always works with the latest state
setCount(prevCount => prevCount + 1);
```

**[SAM]:** When would the first form be wrong?

**[JORDAN]:** When you call `setCount` multiple times in rapid succession, or in async code. React batches state updates and `count` in a closure might be stale by the time your update runs. The functional form always gets the most current value from React's internal queue. Golden rule: if your new state depends on the old state, use the functional form.

**📝 Best Practice:** Keep state minimal. Don't store derived data in state. If you can calculate something from existing state or props, calculate it — don't duplicate it into another piece of state.

```jsx
const [firstName, setFirstName] = useState("Sam");
const [lastName, setLastName] = useState("Smith");

// ❌ Don't do this
const [fullName, setFullName] = useState("Sam Smith");

// ✅ Derive it
const fullName = `${firstName} ${lastName}`;
```

---

# SEGMENT 6: Handling Events `(62:00 – 76:00)`

**[SAM]:** We've been using `onClick` without really explaining it. Let's go deeper.

**[JORDAN]:** Good call. In the browser world, events are things that happen: a user clicks something, types something, moves their mouse, submits a form, presses a key. The browser fires an event, and you can attach a **handler** — a function — to respond to it.

In plain JavaScript, you'd write: `document.getElementById('myButton').addEventListener('click', handleClick)`. In React, you do it directly in JSX:

```jsx
function MyButton() {
  function handleClick() {
    alert("You clicked me!");
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

**[SAM]:** Notice it's `onClick` not `onclick` — the camelCase thing again.

**[JORDAN]:** Right. All event handlers in JSX are camelCase: `onClick`, `onChange`, `onSubmit`, `onKeyDown`, `onMouseEnter`, `onFocus`, `onBlur`. They map to standard browser events, just with React's camelCase naming convention.

**[SAM]:** What's the event object? I see code like `(e) => ...`

**[JORDAN]:** When an event fires, the browser creates an event object that contains information about what happened. React wraps this in a synthetic event for cross-browser consistency. You get it as the first argument to your handler:

```jsx
function SearchInput() {
  function handleChange(e) {
    console.log(e.target.value); // what the user typed
  }

  return <input type="text" onChange={handleChange} />;
}
```

`e.target` is the DOM element that fired the event. `e.target.value` is the current value of an input field. `e.preventDefault()` prevents the browser's default behavior — crucial for forms.

**[SAM]:** What are the most commonly used events?

**[JORDAN]:** Let me give you the core set:

| Event | Fired when | Common use |
|-------|------------|------------|
| `onClick` | Element is clicked | Buttons, links |
| `onChange` | Input value changes | Text fields, selects, checkboxes |
| `onSubmit` | Form is submitted | Form handling |
| `onKeyDown` | Key is pressed down | Keyboard shortcuts |
| `onFocus` / `onBlur` | Element gains/loses focus | Validation, tooltips |
| `onMouseEnter` / `onMouseLeave` | Mouse enters/leaves | Hover effects |

**[SAM]:** Now I've seen people write `onClick={handleClick}` and also `onClick={() => doSomething()}`. When do you use which?

**[JORDAN]:** This is a really common point of confusion. If you write `onClick={handleClick}`, you're passing the *function reference itself* — React will call it when clicked. If you write `onClick={() => handleClick()}`, you're passing an *inline arrow function* that calls `handleClick` when executed.

The gotcha: **never call the function in the JSX itself**.

```jsx
// ❌ WRONG — calls handleClick immediately during render, not on click
<button onClick={handleClick()}>

// ✅ Reference — React calls it on click
<button onClick={handleClick}>

// ✅ Arrow function — React calls the arrow function on click, which calls handleClick
<button onClick={() => handleClick()}>
```

**[SAM]:** That `handleClick()` with parentheses mistake seems like it would bite everyone at least once.

**[JORDAN]:** Every single beginner. At least once. The rule: in JSX, `onClick={fn}` not `onClick={fn()}`.

**[SAM]:** When would you actually *need* the arrow function form?

**[JORDAN]:** When you need to pass arguments. You can't pass arguments to a plain function reference without calling it. The arrow function wraps the call:

```jsx
function ItemList({ items }) {
  function deleteItem(itemId) {
    // delete logic
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => deleteItem(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

**[SAM]:** Makes sense. What about event propagation? Like if I click a button inside a div, does the div's onClick also fire?

**[JORDAN]:** Yes — that's event bubbling. Events bubble up through the DOM tree. A click on a button fires the button's `onClick`, then the parent div's `onClick`, then the grandparent's, all the way to the document. If you want to stop it:

```jsx
function handleButtonClick(e) {
  e.stopPropagation(); // stops the event from bubbling further
  // handle button click
}
```

**📝 Pitfall:** Forgetting `e.preventDefault()` on form submit handlers. Without it, the browser will try to navigate to the form's action URL — the default HTML form behavior — which in a React app means a full page reload and loss of all your state.

```jsx
function LoginForm() {
  function handleSubmit(e) {
    e.preventDefault(); // CRITICAL — prevents page reload
    // process form data
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

---

# SEGMENT 7: Conditional Rendering `(76:00 – 88:00)`

**[SAM]:** Okay let's talk about showing and hiding things. Like a loading spinner while data is fetching, or an error message if something goes wrong.

**[JORDAN]:** Conditional rendering. Since JSX is just JavaScript expressions, you use JavaScript conditions to decide what to render. The three main patterns are: the `if` statement with early return, the ternary operator, and the `&&` operator.

**[SAM]:** Let's go through each.

**[JORDAN]:** Pattern one: **early return**. The cleanest option when you want to render completely different things based on a condition.

```jsx
function UserProfile({ isLoading, error, user }) {
  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (error) {
    return <ErrorMessage message={error} />;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

Read it like a stack of guards. Handle the exceptional cases first, then render the happy path at the bottom. This is the most readable pattern for complex conditions.

**[SAM]:** Pattern two?

**[JORDAN]:** The **ternary operator**: `condition ? valueIfTrue : valueIfFalse`. Use it when you want one of two things inline within your JSX:

```jsx
function StatusBadge({ isOnline }) {
  return (
    <span className={isOnline ? "badge-green" : "badge-grey"}>
      {isOnline ? "Online" : "Offline"}
    </span>
  );
}
```

**[SAM]:** Clean. What about when there's no "else" — just show something or nothing?

**[JORDAN]:** That's pattern three: the **`&&` short-circuit**. In JavaScript, `condition && expression` evaluates to the expression if condition is truthy, and to `false` (or the left side) if condition is falsy. React renders nothing for `false`, `null`, or `undefined`.

```jsx
function Notification({ hasUnread, count }) {
  return (
    <div>
      <BellIcon />
      {hasUnread && <span className="badge">{count}</span>}
    </div>
  );
}
```

If `hasUnread` is false, the badge doesn't render. If it's true, the badge renders.

**[SAM]:** What's the pitfall here?

**[JORDAN]:** The `&&` trap with numbers. This is a very common bug:

```jsx
// ❌ BUG — renders "0" on screen when count is 0!
{count && <Badge count={count} />}

// ✅ Correct — explicitly check for truthiness
{count > 0 && <Badge count={count} />}

// ✅ Also correct
{!!count && <Badge count={count} />}
```

**[SAM]:** Why does it render "0"?

**[JORDAN]:** Because `0 && anything` evaluates to `0` in JavaScript — the number zero. And React *does* render `0` to the screen because it's a valid JSX expression (a number). But `false && anything` evaluates to `false`, which React doesn't render. It's a subtle JavaScript quirk. Always use a boolean condition with `&&`, not a potentially-zero number.

**[SAM]:** Good to know. Can you nest ternaries?

**[JORDAN]:** You *can* but you almost certainly *shouldn't*. Nested ternaries become unreadable very quickly:

```jsx
// ❌ Hard to read
return status === 'loading' ? <Spinner /> : status === 'error' ? <Error /> : <Content />;

// ✅ Much clearer with early returns
if (status === 'loading') return <Spinner />;
if (status === 'error') return <Error />;
return <Content />;
```

**📝 Best Practice:** Prefer early returns for complex conditions, ternary for simple two-option inline choices, `&&` for "show or nothing." If your conditional rendering logic gets complex, extract it into a separate component.

---

# SEGMENT 8: Lists and Keys `(88:00 – 100:00)`

**[SAM]:** Almost every real UI has a list — a list of messages, a table of users, a grid of products. How does React handle rendering a collection of items?

**[JORDAN]:** With JavaScript's `.map()` method. Since JSX accepts any JavaScript expression inside curly braces, you can map over an array and return JSX for each element:

```jsx
function FruitList({ fruits }) {
  return (
    <ul>
      {fruits.map(fruit => (
        <li>{fruit}</li>
      ))}
    </ul>
  );
}

// Usage:
<FruitList fruits={["apple", "banana", "cherry"]} />
```

**[SAM]:** That generates three `<li>` elements. Clean.

**[JORDAN]:** But if you run this, React will warn you in the console: "Each child in a list should have a unique 'key' prop." This is the key concept — pun intended.

**[SAM]:** What's a key?

**[JORDAN]:** A `key` is a special prop you must add to each element when rendering a list. It tells React which item is which, so when the list changes, React can figure out what was added, removed, or moved — without re-rendering everything from scratch.

```jsx
function FruitList({ fruits }) {
  return (
    <ul>
      {fruits.map(fruit => (
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  );
}
```

**[SAM]:** Why does React need this? Can't it just diff the list the same way it diffs everything else?

**[JORDAN]:** This is the deep question. Without keys, React identifies list items by their *position*. If you have three items and you remove the second one, React sees item at position 0 is the same, position 1 changed, position 2 is gone — and it updates and removes accordingly. That seems fine, until you have items with their own state.

Imagine a list of text inputs. Each input has some text typed into it. Now you remove the first item. Without keys, React sees item 0 is different, item 1 is different, and re-renders both inputs. The typed values might end up in the wrong inputs. With keys, React knows exactly which input is which by identity, not by position, so it removes the right one and leaves the others untouched.

**[SAM]:** So keys are for React's bookkeeping, not for anything visible in the UI.

**[JORDAN]:** Exactly. They never appear in the rendered HTML. They're a hint to React's reconciliation algorithm.

**[SAM]:** What should I use as a key?

**[JORDAN]:** Use a stable, unique identifier from your data — usually a database ID or a UUID:

```jsx
{users.map(user => (
  <UserCard key={user.id} user={user} />
))}
```

**[SAM]:** What about using the array index as the key? I've seen that.

**[JORDAN]:** A very common mistake. Don't use array index as a key if the list can be reordered, filtered, or have items removed from the middle. When positions change, React gets confused:

```jsx
// ❌ Fragile — index changes when items are added/removed
{items.map((item, index) => (
  <Item key={index} item={item} />
))}

// ✅ Stable — uses the item's own identity
{items.map(item => (
  <Item key={item.id} item={item} />
))}
```

**[SAM]:** When is index okay to use?

**[JORDAN]:** Only when the list is completely static — it never reorders, filters, or has items inserted or removed. Even then, having a proper ID is always safer. If your data genuinely has no IDs, generate them when you create the data, not at render time. Never do `key={Math.random()}` — that generates a new key every render and defeats the entire purpose.

**[SAM]:** What about nested lists?

**[JORDAN]:** Keys only need to be unique *among siblings* at the same level, not globally across the entire app. So the same ID can appear in different lists.

**📝 Pitfall:** Forgetting that keys must be on the *outermost* element returned from `.map()`, not buried inside:

```jsx
// ❌ Wrong — key is on an inner element
{items.map(item => (
  <div>
    <li key={item.id}>{item.name}</li>
  </div>
))}

// ✅ Correct — key is on the root returned element
{items.map(item => (
  <div key={item.id}>
    <li>{item.name}</li>
  </div>
))}
```

---

# SEGMENT 9: useEffect — Talking to the Outside World `(100:00 – 116:00)`

**[SAM]:** We've been building pure UI components so far. But real apps need to do things — fetch data from an API, set up a timer, subscribe to a WebSocket, update the browser title. Where does that fit in?

**[JORDAN]:** All of those are **side effects**. A side effect is anything your component does that reaches *outside* the pure function world of rendering — network requests, timers, subscriptions, directly manipulating the DOM, local storage. React gives you the `useEffect` hook to handle these.

**[SAM]:** Why do they need special treatment? Why not just call `fetch` directly in the component body?

**[JORDAN]:** Because component functions run on every render. If you call `fetch` directly in the function body, you'd make a network request every single time the component re-renders — which could be many times, including for reasons completely unrelated to the data you're fetching. `useEffect` lets you control *when* the side effect runs.

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/users/${userId}`)
      .then(response => response.json())
      .then(data => {
        setUser(data);
        setIsLoading(false);
      });
  }, [userId]);

  if (isLoading) return <Spinner />;
  return <div>{user.name}</div>;
}
```

**[SAM]:** Let me break that down. `useEffect` takes a function — the effect — and an array. That array is the dependency array?

**[JORDAN]:** Yes. The **dependency array** is how you tell React when to run the effect. The rules are:

- **No dependency array**: run after every render. Usually wrong.
- **Empty array `[]`**: run only once, after the component first mounts. Good for one-time setup.
- **Array with values `[userId]`**: run after the first render, and again whenever `userId` changes. This is what we want for fetching by ID.

**[SAM]:** So in our example, every time the parent passes a different `userId`, the effect re-runs and fetches the new user?

**[JORDAN]:** Exactly. React compares the dependency values between renders. When `userId` changes, the effect runs again. This is the core pattern for data fetching.

**[SAM]:** What goes in the dependency array?

**[JORDAN]:** Every reactive value your effect uses — every state variable, every prop, every variable defined inside the component that you reference inside the effect. If you use `userId` inside the effect, `userId` goes in the array.

**[SAM]:** What if I forget to include something?

**[JORDAN]:** You get a stale closure bug. Your effect is using an old snapshot of the value. React has an ESLint rule — `react-hooks/exhaustive-deps` — that catches this automatically. Always have this rule enabled.

**[SAM]:** What's cleanup? I've seen effects return a function.

**[JORDAN]:** Some side effects need to be undone when the component unmounts or before the effect runs again. You return a cleanup function from your effect:

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    setSeconds(prev => prev + 1);
  }, 1000);

  // Cleanup: clear the timer when component unmounts
  return () => clearInterval(timer);
}, []);
```

Without the cleanup, the timer would keep running even after the component is gone, calling `setSeconds` on an unmounted component — that's a memory leak and React will warn you about it.

**[SAM]:** Other cases where you need cleanup?

**[JORDAN]:** Event listeners you added manually, WebSocket connections, RxJS subscriptions, AbortController for fetch requests. Anything you "start" in an effect, you should "stop" in the cleanup.

Here's a fetch with proper cleanup using AbortController:

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setUser)
    .catch(err => {
      if (err.name !== 'AbortError') setError(err.message);
    });

  return () => controller.abort(); // cancel the request if userId changes
}, [userId]);
```

**[SAM]:** That's the AbortController pattern for cancelling in-flight requests. Smart.

**[JORDAN]:** Critical in production. Without it, if `userId` changes quickly — think a typeahead search — you can have multiple in-flight requests, and the older slower one might return *after* the newer one, showing stale data. The abort ensures old requests are cancelled before new ones start.

**📝 The most common useEffect mistakes:**

```jsx
// ❌ Infinite loop — effect sets state, re-render triggers effect again
useEffect(() => {
  setCount(count + 1); // this changes count, which triggers the effect, which changes count...
}, [count]);

// ❌ Missing dependency — stale closure on userId
useEffect(() => {
  fetchUser(userId); // uses userId but doesn't declare it
}, []); // ESLint will catch this

// ❌ Async function directly as the effect (useEffect doesn't accept async)
useEffect(async () => { // wrong!
  const data = await fetch('/api/data');
}, []);

// ✅ Correct: define async inside the effect
useEffect(() => {
  async function loadData() {
    const res = await fetch('/api/data');
    const data = await res.json();
    setData(data);
  }
  loadData();
}, []);
```

**[SAM]:** That async gotcha is interesting. Why can't you make the effect itself async?

**[JORDAN]:** `useEffect`'s callback is supposed to return either nothing or a cleanup function. An async function returns a Promise, not a cleanup function. React would receive a Promise where it expects a function or undefined, and the cleanup mechanism would break entirely.

**📝 Best Practice:** When data fetching becomes complex, consider libraries like TanStack Query (formerly React Query) or SWR. They handle loading states, error states, caching, refetching, and cancellation for you — so you don't have to roll it all by hand.

---

# SEGMENT 10: useRef — Remembering Without Re-rendering `(116:00 – 126:00)`

**[SAM]:** We've got useState for reactive values that trigger re-renders. But what if I need to remember something between renders without causing a re-render?

**[JORDAN]:** That's `useRef`. Think of a ref as a box — an object with a `.current` property — that persists across renders. Reading or writing `.current` does not trigger a re-render.

```jsx
import { useRef } from 'react';

function StopwatchComponent() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);

  function start() {
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  }

  function stop() {
    clearInterval(intervalRef.current);
  }

  return (
    <div>
      <p>{seconds}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

**[SAM]:** So the interval ID is stored in a ref — it persists between renders, but changing it doesn't cause unnecessary re-renders.

**[JORDAN]:** Perfect. The other major use case for `useRef` is accessing DOM elements directly — for cases where React's declarative model isn't enough. Focusing an input, measuring an element's size, integrating with a third-party library that needs a real DOM node.

```jsx
function SearchBar() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus(); // direct DOM call
  }

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
    </div>
  );
}
```

By passing a ref to the `ref` prop of a JSX element, React sets `ref.current` to the actual DOM node after mounting.

**[SAM]:** When should I use ref vs state?

**[JORDAN]:** Simple rule:
- **State**: when the value should be *displayed* in the UI, and changes should trigger a re-render.
- **Ref**: when you need to *remember* a value between renders but changing it shouldn't cause re-renders — timers, DOM nodes, previous values, flags.

```jsx
// ✅ State: used in the render output
const [inputValue, setInputValue] = useState("");

// ✅ Ref: used internally, not displayed
const renderCountRef = useRef(0);
renderCountRef.current += 1; // no re-render triggered
```

**📝 Pitfall:** Don't read or write `ref.current` during rendering — that includes inside the component body and inside JSX. Refs are meant for side effects and event handlers. Reading a ref in render is not reactive — React won't know to re-render when it changes.

---

# SEGMENT 11: useMemo & useCallback — Performance Tools `(126:00 – 140:00)`

**[SAM]:** Now let's talk about `useMemo` and `useCallback`. I see these in production code all the time and I never quite understand when to use them.

**[JORDAN]:** These are performance optimization hooks. But first, I want to give you the most important piece of advice about them: **don't reach for them by default.** React is fast. Premature optimization is the root of a lot of complicated, hard-to-maintain React code. Only use these when you have a *measured* performance problem.

**[SAM]:** Understood. But let's understand what they do.

**[JORDAN]:** Let's start with `useMemo`. Remember that every time a component re-renders, the entire function body runs again. If you have an expensive calculation inside, it runs on every render — even when the inputs haven't changed.

```jsx
function ProductList({ products, filterText }) {
  // This filters potentially thousands of products on EVERY render
  const filteredProducts = products.filter(p =>
    p.name.toLowerCase().includes(filterText.toLowerCase())
  );

  return <List items={filteredProducts} />;
}
```

**[SAM]:** If the parent re-renders for some unrelated reason, this filter runs again even though `products` and `filterText` didn't change.

**[JORDAN]:** Exactly. `useMemo` memoizes the result — it caches the computed value and only recalculates when the dependencies change:

```jsx
import { useMemo } from 'react';

function ProductList({ products, filterText }) {
  const filteredProducts = useMemo(() => {
    return products.filter(p =>
      p.name.toLowerCase().includes(filterText.toLowerCase())
    );
  }, [products, filterText]); // only re-runs when these change

  return <List items={filteredProducts} />;
}
```

**[SAM]:** And `useCallback`?

**[JORDAN]:** `useCallback` is the same idea but for *functions* instead of values. Remember how I said everything in a component function is recreated on every render? That includes inline functions. And in JavaScript, a new function created each render is a *different reference* than the previous render's function, even if it does the same thing.

This matters when you pass functions as props to child components that are optimized with `React.memo`.

Let me explain `React.memo` first. It's a wrapper you can put around a component to tell React: "only re-render this component if its props actually changed."

```jsx
const ExpensiveList = React.memo(function ExpensiveList({ items, onItemClick }) {
  console.log("ExpensiveList rendering..."); // we want this rarely
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>{item.name}</li>
      ))}
    </ul>
  );
});
```

**[SAM]:** Good — so it's like a shouldComponentUpdate.

**[JORDAN]:** Similar concept. But here's the problem — if the parent creates `onItemClick` as an inline function:

```jsx
function Parent({ items }) {
  function handleItemClick(id) {
    console.log("clicked", id);
  }

  return <ExpensiveList items={items} onItemClick={handleItemClick} />;
}
```

Every render of `Parent` creates a *new* `handleItemClick` function. `React.memo` compares `onItemClick` before and after — new reference = different prop = `ExpensiveList` re-renders. The memoization is useless.

**[SAM]:** And `useCallback` fixes this?

**[JORDAN]:** Yes — it returns the same function reference across renders, unless the dependencies change:

```jsx
import { useCallback } from 'react';

function Parent({ items }) {
  const handleItemClick = useCallback((id) => {
    console.log("clicked", id);
  }, []); // no dependencies — always the same function

  return <ExpensiveList items={items} onItemClick={handleItemClick} />;
}
```

Now `React.memo` sees the same function reference and skips the re-render.

**[SAM]:** So: `useMemo` for expensive values, `useCallback` for stable function references passed to memoized children.

**[JORDAN]:** Exactly. And the honest truth about when to use them:

```
useMemo:
✅ Expensive calculation (filtering thousands of items, sorting, complex transforms)
✅ Object/array you pass to a memoized child component (prevents unnecessary re-renders)
❌ Simple calculations that take microseconds
❌ Everything by default (premature optimization)

useCallback:
✅ Functions passed as props to components wrapped in React.memo
✅ Functions in useEffect dependency arrays (to prevent infinite loops)
❌ Every function in every component by default
❌ Functions you don't pass anywhere
```

**[SAM]:** What's the cost of using them unnecessarily?

**[JORDAN]:** The memoization itself has overhead — React has to store the cached value, run the comparison, manage the dependency tracking. For simple values, that overhead can actually be *more* expensive than just recalculating. The React team explicitly says: profile first, optimize second.

**📝 Pitfall:** Using `useMemo`/`useCallback` with incorrect dependencies creates hard-to-find bugs. The cached value stays stale because you forgot a dependency. Always let the ESLint exhaustive-deps rule guide you.

---

# SEGMENT 12: useContext — Sharing Data Without Prop Drilling `(140:00 – 152:00)`

**[SAM]:** I want to talk about a pain point I've already experienced mentally just thinking about React. If data flows down through props, and I have something like the current user's info that needs to be available in a header, a sidebar, and deep in a settings page — do I really have to pass it through every component in between?

**[JORDAN]:** You've just described prop drilling, and yes, it's painful. Passing data through five layers of components that don't care about it — they're just passing it down — is messy and brittle. Change the data shape and you have to update every component in the chain.

React's solution is the **Context API**. Context lets you create a value at a high level in your component tree and make it available to any descendant component, without passing it through every level.

**[SAM]:** Like a global variable but scoped to a subtree?

**[JORDAN]:** Good analogy. Let's build it step by step.

**Step 1: Create the context.**

```jsx
import { createContext } from 'react';

const UserContext = createContext(null); // null is the default value
```

**Step 2: Provide the value.** Wrap the part of your tree that needs access in the context's `Provider`:

```jsx
function App() {
  const [currentUser, setCurrentUser] = useState(null);

  return (
    <UserContext.Provider value={currentUser}>
      <Header />
      <Sidebar />
      <MainContent />
    </UserContext.Provider>
  );
}
```

**Step 3: Consume the value.** Any component inside the Provider can use `useContext`:

```jsx
import { useContext } from 'react';

function UserAvatar() {
  const user = useContext(UserContext);

  if (!user) return <GuestIcon />;
  return <img src={user.avatarUrl} alt={user.name} />;
}
```

`UserAvatar` can be nested ten levels deep — it doesn't matter. `useContext` reaches up to the nearest `UserContext.Provider` and gets the value directly.

**[SAM]:** That's clean. What are good use cases for Context?

**[JORDAN]:** Things that are truly global or theme-level:
- Current logged-in user
- UI theme (light/dark mode)
- Locale/language
- Feature flags
- Shopping cart contents

Things that many unrelated components need to know about.

**[SAM]:** What about using Context as a full state management solution?

**[JORDAN]:** Here's where I need to be careful. Context is not a performance-optimized state manager. The key thing to know: **when a Context value changes, every component that calls `useContext` for that context re-renders**, even if it only cares about a part of the value that didn't change.

```jsx
// This context holds both user info and preferences
const AppContext = createContext();

function App() {
  const [user, setUser] = useState({...});
  const [preferences, setPreferences] = useState({...});

  return (
    // Every time EITHER user OR preferences changes,
    // every consumer of this context re-renders
    <AppContext.Provider value={{ user, preferences, setUser, setPreferences }}>
      ...
    </AppContext.Provider>
  );
}
```

**[SAM]:** So if I change the user's avatar, every component subscribed to the context re-renders — even components that only use preferences?

**[JORDAN]:** Exactly. The fix is to split contexts by what changes together:

```jsx
const UserContext = createContext();
const PreferencesContext = createContext();

function App() {
  const [user, setUser] = useState({...});
  const [preferences, setPreferences] = useState({...});

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <PreferencesContext.Provider value={{ preferences, setPreferences }}>
        {children}
      </PreferencesContext.Provider>
    </UserContext.Provider>
  );
}
```

Now components that only care about preferences don't re-render when the user changes.

**📝 When to use Context vs a state management library (Redux, Zustand):**

| | Context | Zustand/Redux |
|---|---|---|
| Data changes rarely | ✅ Great | Overkill |
| Many updates per second | ❌ Re-renders | ✅ Optimized |
| Complex state logic | Messy | ✅ Better structure |
| Small/medium app | ✅ Fine | Probably overkill |
| Large complex app | May struggle | ✅ Better choice |

For most apps: start with Context. If you notice performance issues or the logic becomes complex, reach for Zustand — it's lightweight and excellent.

---

# SEGMENT 13: Custom Hooks — Your Own Reusable Logic `(152:00 – 164:00)`

**[SAM]:** We've covered useState, useEffect, useRef, useMemo, useCallback, useContext. Are there more built-in hooks?

**[JORDAN]:** A few more — `useReducer` for complex state, `useId` for accessibility, `useTransition` for async rendering. But I want to talk about something more powerful: custom hooks. The ability to create *your own* hooks is one of the best features in React.

**[SAM]:** What's a custom hook?

**[JORDAN]:** A custom hook is just a regular JavaScript function whose name starts with `use` and that can call other hooks. That's it. It's a way to extract and reuse stateful logic across multiple components.

**[SAM]:** Give me a concrete example.

**[JORDAN]:** The fetch-data pattern we wrote earlier — loading state, error state, data state, effect with cleanup — you're going to write that in almost every component that loads remote data. Instead of duplicating it everywhere, extract it into a custom hook:

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setIsLoading(true);
    setError(null);

    fetch(url, { signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(data => {
        setData(data);
        setIsLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err.message);
          setIsLoading(false);
        }
      });

    return () => controller.abort();
  }, [url]);

  return { data, isLoading, error };
}
```

Now every component that needs to fetch data can use this hook:

```jsx
function UserProfile({ userId }) {
  const { data: user, isLoading, error } = useFetch(`/api/users/${userId}`);

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  return <div>{user.name}</div>;
}

function ProductPage({ productId }) {
  const { data: product, isLoading } = useFetch(`/api/products/${productId}`);
  // ...
}
```

**[SAM]:** All the fetching logic in one place. Any component that needs data just calls `useFetch`.

**[JORDAN]:** And if you want to add retry logic, caching, or error reporting later — you add it in one place.

**[SAM]:** What's the naming rule? It has to start with `use`?

**[JORDAN]:** Yes — the `use` prefix is a convention that the React linter enforces. It tells React and the linter that this function is a hook and must follow the **Rules of Hooks**:
1. Only call hooks at the top level — never inside if statements, loops, or nested functions
2. Only call hooks from React components or other custom hooks — never from regular JavaScript functions

**[SAM]:** Why no hooks inside if statements?

**[JORDAN]:** React tracks hooks by their call order. Every render, hooks must be called in exactly the same order. If you conditionally call a hook, the order changes between renders and React loses track of which state belongs to which hook:

```jsx
// ❌ NEVER do this
function BadComponent({ isAdmin }) {
  if (isAdmin) {
    const [adminData, setAdminData] = useState(null); // conditional hook!
  }
  const [userData, setUserData] = useState(null);
}

// ✅ Hook at the top level, use the value conditionally
function GoodComponent({ isAdmin }) {
  const [adminData, setAdminData] = useState(null);
  const [userData, setUserData] = useState(null);

  // Conditional logic goes here, not around the hook call
  useEffect(() => {
    if (isAdmin) {
      fetchAdminData().then(setAdminData);
    }
  }, [isAdmin]);
}
```

**📝 Good custom hooks to build:**
- `useLocalStorage(key, defaultValue)` — state synced to localStorage
- `useDebounce(value, delay)` — debounced value for search inputs
- `useWindowSize()` — reactive window dimensions
- `useOnClickOutside(ref, handler)` — detect clicks outside a modal
- `usePrevious(value)` — access the previous render's value
- `useToggle(initialValue)` — boolean state with toggle function

---

# SEGMENT 14: Component Composition Patterns `(164:00 – 176:00)`

**[SAM]:** Let's talk about how to structure components that need to be flexible. Like, how do I build a `<Modal>` component that can contain different things in different places?

**[JORDAN]:** This is component composition — the art of building flexible, reusable components without overcomplicating them. The foundation is the `children` prop.

**[SAM]:** The children prop?

**[JORDAN]:** Every React component automatically receives a special `children` prop containing whatever you put *between* its opening and closing tags:

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

// Using it:
<Card>
  <h2>My Title</h2>
  <p>Some content here.</p>
  <button>Action</button>
</Card>
```

The `Card` component doesn't know or care what its children are — it just renders them inside the card structure. This makes `Card` incredibly reusable.

**[SAM]:** That's clean. What about components that need multiple "slots" — like a card with a separate header, body, and footer?

**[JORDAN]:** You use named props for additional slots. There's nothing special about the name `children` — it's just a convention for the "main" content. You can pass JSX as any prop:

```jsx
function Modal({ title, children, footer }) {
  return (
    <div className="modal-overlay">
      <div className="modal">
        <div className="modal-header">
          <h2>{title}</h2>
        </div>
        <div className="modal-body">
          {children}
        </div>
        <div className="modal-footer">
          {footer}
        </div>
      </div>
    </div>
  );
}

// Using it:
<Modal
  title="Confirm Delete"
  footer={
    <>
      <button onClick={cancel}>Cancel</button>
      <button onClick={confirm}>Delete</button>
    </>
  }
>
  <p>Are you sure you want to delete this item?</p>
</Modal>
```

**[SAM]:** You're passing JSX as a prop. Mind-bending at first but it makes sense.

**[JORDAN]:** JSX is just a value — you can pass it anywhere you'd pass any other value. This is what makes React composition so flexible.

**[SAM]:** What about the render props pattern? I've heard that term.

**[JORDAN]:** Render props is a pattern where a component accepts a *function* as a prop, and that function returns JSX. It's used when the component needs to provide some data or behavior to the thing it renders:

```jsx
function DataProvider({ render }) {
  const [data, setData] = useState(null);
  // ...fetch data, manage state...

  return render(data); // calls the function and renders the result
}

// Using it:
<DataProvider render={(data) => (
  <UserList users={data} />
)} />
```

**[SAM]:** When would I use that vs a custom hook?

**[JORDAN]:** Modern React: reach for custom hooks first. They're cleaner and more composable. Render props are mostly a pattern you'll encounter in older codebases. But they still have niche uses — particularly for libraries like React Motion or Downshift.

**📝 Best Practice:** Favor composition over configuration. Instead of a component with a dozen boolean props (`showHeader`, `showFooter`, `hasCloseButton`, `isFullScreen`), build small focused components and compose them. The consumer has full control without the component needing to anticipate every possible variant.

---

# SEGMENT 15: Forms and Controlled Inputs `(176:00 – 188:00)`

**[SAM]:** Forms are everywhere. Login forms, settings pages, search boxes. React seems to handle them differently than plain HTML.

**[JORDAN]:** There are two approaches in React: **controlled** and **uncontrolled** components. In plain HTML, form inputs manage their own state internally — you submit the form and the browser collects the values. React gives you an alternative: make React the source of truth for every keystroke.

**[SAM]:** What does controlled mean?

**[JORDAN]:** A controlled input is one where React state drives the displayed value, and every user keystroke updates that state:

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    console.log({ email, password });
    // Call your API here
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

**[SAM]:** So `value={email}` makes React control what's displayed, and `onChange` updates the state, which updates the displayed value. It's a loop.

**[JORDAN]:** Exactly. React renders the input with the value from state. User types. `onChange` fires. We call `setEmail` with the new value. State updates. React re-renders the input with the new value. It happens so fast it feels instant, but it's a controlled loop every keystroke.

**[SAM]:** What does this buy me over just letting the browser handle it?

**[JORDAN]:** Real-time validation, conditional UI, formatted inputs, derived values. If the email state updates on every keystroke, you can show "Please enter a valid email" in real time. You can disable the submit button if the form is incomplete. You can format a phone number as the user types.

**[SAM]:** And uncontrolled?

**[JORDAN]:** Uncontrolled inputs let the DOM manage the state. You read the value only when you need it — usually on submit — using a ref:

```jsx
function SimpleForm() {
  const nameRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    console.log(nameRef.current.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" ref={nameRef} defaultValue="Sam" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**[SAM]:** When would I use uncontrolled?

**[JORDAN]:** File inputs — you can't control the value of a file input for security reasons, so refs are the only option. Also simple forms where you don't need any real-time behavior. For most forms though: controlled.

**📝 Form management libraries:**

For complex forms — many fields, complex validation, dynamic field arrays — consider React Hook Form. It gives you performant controlled forms with minimal re-renders, built-in validation, and excellent TypeScript support.

```jsx
import { useForm } from 'react-hook-form';

function SignupForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  function onSubmit(data) {
    console.log(data);
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email", { required: true, pattern: /^\S+@\S+$/ })} />
      {errors.email && <p>Valid email required</p>}
      <button type="submit">Sign up</button>
    </form>
  );
}
```

---

# SEGMENT 16: React Router — Navigation Basics `(188:00 – 198:00)`

**[SAM]:** A real app has multiple pages — home page, product page, user settings. How does navigation work in React?

**[JORDAN]:** React itself only handles UI rendering — it has no built-in concept of pages or routing. Navigation is handled by a library called **React Router** (v6 is current). It lets you map URL paths to different components.

**[SAM]:** How does it work?

**[JORDAN]:** You wrap your app in a router, define routes, and React Router renders the matching component based on the current URL:

```jsx
import { BrowserRouter, Routes, Route, Link, useParams } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/products">Products</Link>
      </nav>

      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/products" element={<ProductsPage />} />
        <Route path="/products/:productId" element={<ProductDetailPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**[SAM]:** The `:productId` in the path is a URL parameter?

**[JORDAN]:** Yes. And you read URL parameters using the `useParams` hook:

```jsx
function ProductDetailPage() {
  const { productId } = useParams();
  const { data: product } = useFetch(`/api/products/${productId}`);

  return <div>{product?.name}</div>;
}
```

**[SAM]:** And `<Link>` instead of `<a>`?

**[JORDAN]:** Critical distinction. Always use `<Link>` for internal navigation in React apps. A regular `<a href="/about">` causes a full page reload — the browser makes a new request, the app starts over from scratch, all state is lost. `<Link>` does client-side navigation — the URL changes, React Router renders the matching component, no page reload, state is preserved.

**[SAM]:** What about navigating programmatically — like redirecting after login?

**[JORDAN]:** Use the `useNavigate` hook:

```jsx
import { useNavigate } from 'react-router-dom';

function LoginPage() {
  const navigate = useNavigate();

  async function handleLogin(credentials) {
    await loginUser(credentials);
    navigate('/dashboard'); // redirect after login
  }
}
```

**[SAM]:** Nested routes?

**[JORDAN]:** React Router v6 supports nested routes — where a parent route renders a layout and child routes render into a specific slot within it. Use the `<Outlet>` component as the placeholder:

```jsx
function DashboardLayout() {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>
        <Outlet /> {/* child route renders here */}
      </main>
    </div>
  );
}

// In your routes:
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} />
  <Route path="settings" element={<SettingsPage />} />
  <Route path="users" element={<UsersPage />} />
</Route>
```

**[SAM]:** So `/dashboard` renders the layout, `/dashboard/settings` renders the layout with `<SettingsPage>` in the Outlet.

**[JORDAN]:** Exactly. Nested routes are great for complex app layouts where different pages share the same shell.

---

# SEGMENT 17: Best Practices, Pitfalls & Mental Models `(198:00 – 210:00)`

**[SAM]:** Let's round out with the big picture — the lessons it takes people years to internalize.

**[JORDAN]:** The collected wisdom. Let me go through the most important ones.

---

**Mental Model 1: Think in components, not pages.**

Coming from a backend background, you might think in terms of pages: the login page, the dashboard page. React thinking is different. Think of your UI as a tree of small, focused components. A "page" is just a component that composes many smaller components. The smaller and more focused each component, the more reusable and testable.

---

**Mental Model 2: State lives as high as it needs to — no higher.**

State should live in the *lowest common ancestor* of all components that need it. If only one component needs a piece of state, keep it there. If two sibling components need the same state, lift it to their parent. If many distant components need it, use Context. Don't reach for global state by default.

```
Before lifting: each component has its own state
After lifting: parent owns state, passes down via props

          Parent (owns state)
         /         \
    Child A     Child B
(reads state)  (modifies state via callback)
```

---

**Mental Model 3: Re-renders are cheap. Unnecessary ones are a symptom, not always a problem.**

Beginners often try to prevent all re-renders with useMemo and useCallback everywhere. Don't. React's reconciliation is fast. A re-render doesn't mean a DOM update — it means the component function ran again and React compared the output. If the DOM doesn't need to change, nothing visible happens. Optimize when you measure a real problem, not before.

---

**Top 10 Pitfalls:**

**1. Mutating state directly.**
```jsx
// ❌ Never
state.items.push(newItem);
// ✅ Always create new
setState(prev => ({ ...prev, items: [...prev.items, newItem] }));
```

**2. Calling the function in JSX instead of passing it.**
```jsx
// ❌ Calls immediately on every render
<button onClick={handleClick()}>
// ✅ Passes the reference
<button onClick={handleClick}>
```

**3. Using index as key for dynamic lists.**
```jsx
// ❌
{items.map((item, i) => <Item key={i} />)}
// ✅
{items.map(item => <Item key={item.id} />)}
```

**4. Missing dependency array values in useEffect.**
```jsx
// ❌ userId is used but not listed — stale closure
useEffect(() => { fetchUser(userId); }, []);
// ✅
useEffect(() => { fetchUser(userId); }, [userId]);
```

**5. The && trap with zero.**
```jsx
// ❌ Renders "0" when count is 0
{count && <Badge />}
// ✅
{count > 0 && <Badge />}
```

**6. Async directly in useEffect.**
```jsx
// ❌
useEffect(async () => { await fetchData(); }, []);
// ✅
useEffect(() => { async function load() { await fetchData(); } load(); }, []);
```

**7. Derived state stored in state.**
```jsx
// ❌ Two pieces of state that must stay in sync
const [items, setItems] = useState([]);
const [itemCount, setItemCount] = useState(0);
// ✅ Derive it
const [items, setItems] = useState([]);
const itemCount = items.length; // just a variable
```

**8. Conditional hooks.**
```jsx
// ❌ NEVER
if (condition) { useState(null); }
// ✅ Hook at top level, use value conditionally
const [value, setValue] = useState(null);
if (condition) { /* use value */ }
```

**9. Not providing cleanup in useEffect.**
```jsx
// ❌ Memory leak — interval runs forever
useEffect(() => {
  const id = setInterval(doThing, 1000);
}, []);
// ✅ Clean up
useEffect(() => {
  const id = setInterval(doThing, 1000);
  return () => clearInterval(id);
}, []);
```

**10. Overusing useEffect for data synchronization.**
```jsx
// ❌ Using useEffect to keep state in sync with other state
const [a, setA] = useState(0);
const [b, setB] = useState(0);
useEffect(() => { setB(a * 2); }, [a]); // unnecessary

// ✅ Derive it directly
const [a, setA] = useState(0);
const b = a * 2; // just calculate it
```

---

**[SAM]:** That last one is interesting. A lot of people would write that useEffect pattern.

**[JORDAN]:** It's extremely common and it's a pattern the React team explicitly calls out as an antipattern. If you find yourself using `useEffect` to sync one piece of state with another, ask yourself: can I calculate the second value from the first? If yes, don't use state for it at all.

**[SAM]:** What are the most important things to learn next after today?

**[JORDAN]:** I'd recommend this sequence:

**Immediate next steps:**
1. **TypeScript with React** — add type safety. All serious React projects use it.
2. **A proper build tool** — Vite is the modern standard for React apps. Fast dev server, optimal builds.
3. **React DevTools** — the browser extension that lets you inspect your component tree, see state and props, and profile renders.

**Within the first month:**
4. **TanStack Query** — for data fetching. Replaces most `useEffect` + `useState` for API calls.
5. **React Hook Form** — for forms.
6. **Testing** — React Testing Library is the standard. Test behavior, not implementation.

**When you're ready:**
7. **Next.js** — the most popular React framework. Adds server-side rendering, file-based routing, API routes, and excellent deployment tooling. Most new React projects start here, not with plain React.
8. **Zustand or Redux Toolkit** — for state management at scale.

---

**[SAM]:** Final reflection: what's the one thing you wish someone had told you when you started React?

**[JORDAN]:** React is fundamentally about this: **your UI is a pure function of your state**. Every time I feel lost or confused in React, I come back to that. I ask myself: "what is my state? What should the UI look like for this state? Why doesn't the actual UI match what I expect?" Ninety percent of React bugs reduce to one of: the wrong value is in state, state was mutated instead of replaced, a re-render wasn't triggered, or an effect had the wrong dependencies. Everything else is variation on those four themes.

If you keep the "UI is a function of state" mental model in mind, React stops feeling magical and starts feeling deeply logical.

**[SAM]:** That's a genuinely useful compass. Jordan, this has been incredible. Thank you for starting from the very beginning and not skipping any steps.

**[JORDAN]:** That's the only way to really learn this. I'll see you in the next episode.

🎵 *Closing music fades in and out*

---

## 📚 Quick Reference Card

### Hooks at a Glance

| Hook | Purpose | When to use |
|------|---------|------------|
| `useState` | Reactive state | Any value that should trigger re-render when changed |
| `useEffect` | Side effects | Fetching, timers, subscriptions, DOM manipulation |
| `useRef` | Mutable ref / DOM access | Storing values without re-renders, accessing DOM nodes |
| `useMemo` | Memoize expensive values | Costly computations, stable object references for memo |
| `useCallback` | Memoize functions | Stable callback references for memoized children |
| `useContext` | Consume context | Accessing shared values without prop drilling |
| `useReducer` | Complex state logic | Multiple related state values, complex update logic |

### One-Way Data Flow Summary

```
Parent Component
    │
    │  props (data flows down)
    ▼
Child Component
    │
    │  callbacks (events flow up)
    ▼
Parent Component handles it
```

### Rules of Hooks
1. Only call hooks at the **top level** (not inside if/for/while)
2. Only call hooks from **React components** or **custom hooks**
3. Custom hooks must start with **`use`**

### Key Distinction: State vs Props

| | State | Props |
|--|-------|-------|
| Who owns it? | The component itself | The parent component |
| Who changes it? | The component (via setter) | The parent |
| Triggers re-render? | Yes — when setter is called | Yes — when parent passes new value |
| Mutable by receiver? | Only via setter | Never |

---

*Component Thinking is produced for developers who want to build real-world UIs with confidence. All code examples are React 18+ with functional components and hooks.*
