---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: /assets/intro.jpg
# some information about your slides (markdown enabled)
title: Software Development | Foundations
info: |
  ## Software Development | Foundations
# apply unocss classes to the current slide
class: text-left
drawings:
  persist: false
transition: slide-left
mdc: true
---

# React: Optimizations
Frontend Development: Unit 06 - Lesson 08

- [ ] useRef, Good Practices
- [ ] memo, useMemo, useCallback
- [ ] React Dev Tools > Profiler

<div class="abs-br m-6 text-xl">
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
-->


---
transition: slide-left
layout: two-cols
---

# Recap
Q: When count changes, what gets re-rendered?

```jsx
function App() {
  return (
    <>
      <Counter />
      <footer>&copy; 2025</footer>
    </>
  );
}

const H1: React.FC<H1Props> = ({ count }) => {
  return <h1>{count}</h1>;
};

const H2 = () => {
  return <h2>👋</h2>;
};
```

::right::

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <>
      <div className="card">
        <H1 count={count} />
        <H2 />
        <Button
          label="count is"
          badge={count}
          clickHandler={() => setCount(count + 1)}
          severity="secondary"
        />
      </div>
    </>
  );
};
```

---
transition: slide-left
---

# Best Practice: How to organize your code 

1. Constants / Static Values / Default Props / Config
    - Any plain constants, static data, or config variables used inside the component.
    - These should be outside the component if they don’t depend on props/state.
2. React State Hooks (useState)
    - Declare all state variables near the top.
3. Memoization Hooks (useMemo, useCallback)
    - This ensures you use fresh state values as dependencies.
4. Side Effects (useEffect, useLayoutEffect)
    - This order makes it easier to see effects triggered by state changes.
5. Event Handlers and other functions
    - Can come here if they are not memoized, or sometimes before memoized ones if simpler.
    - Often handlers are memoized with useCallback and placed earlier.
6. Rendering Logic / JSX

---
transition: slide-left
---

# useRef

- Recall that JSX is not HTML.  JSX produces JS which in turns manipulates the DOM
- But sometimes we need access to the lower level DOM nodes
- Instead of using `document.querySelector()` to grab DOM elements, we can use React's `useRef`
- `useRef` are mostly used to hold DOM nodes although it can hold any kind of value
- `useRef` can also be used to store a mutable value that persists across renders but doesn't trigger a re-render

```jsx
const FocusInput = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus(); // Auto-focus when component mounts
  }, []); // Empty dependency array means this runs once on mount

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="I'm focused on load" />
    </div>
  );
};
```

---
transition: slide-left
---

# Performance Issues: Unnecessary Re-renders

```jsx
const H1: React.FC<H1Props> = ({ count }) => { return <h1>{count}</h1> };
const H2 = () => { return <h2>👋</h2> };

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <>
      <div className="card">
        <H1 count={count} />
        <H2 /> {/*  SHOULD THIS RE-RENDER EVEN THOUGH NOTHING ABOUT THIS COMPONENT CHANGED? */}
        <Button label="count is" badge={count} clickHandler={() => setCount((count) => count + 1)} 
          severity="secondary" />
      </div>
    </>
  );
};

function Lesson13() {
  return (
    <>
      <Counter />
      <footer>&copy; 2025</footer>
    </>
  );
}
```

---
transition: slide-left
---

# Optimization: React.memo

- Q: Why did H2 re-render if it didn't depend on `count`?
  - A: because React tries to re-render all descendants, regardless.  This is because it's hard to know with 100% certainty, whether H2 depends indirectly on `count`.  
  - React would rather err on the side of caution and re-render all descendants to match UI with state.
- Q: In larger apps, is it possible for one stage change to degrade performance while re-rendering all its children components?
- *ONLY IF you do notice performance issues*, then perhaps using `React.memo()` will help
   - it's a utility that memoizes a component and prevents it from re-rendering if its props/state haven't changed
- Replace our `<H2 />` component with our memoized `<H2_v2 />` Does emoji still re-render?
```jsx
import { memo } from "react";
   ...
const H2_v2 = memo(H2);
   ...
<H2_v2 />
```

---
transition: slide-left
---

# Optimization: useMemo

- Allows us to memoize/remember a computed value between renders which

```jsx
  const [number, setNumber] = useState(1);
  const [renderCount, setRenderCount] = useState(0);

  const factorial = useMemo(() => {
    console.log('Calculating factorial')
    const calculate = (n) => {
      if (n <= 0) return 1;
      return n * calculate(n - 1);
    };
    return calculate(number);
  }, [number]);

  return (
    <div>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(parseInt(e.target.value))}
      />
      <p>Factorial of {number} is {factorial}</p>
      <button onClick={() => setCounter(counter + 1)}>Re-render</button>
      <p>Render count: {renderCount}</p>
    </div>
  );
}
```

---
transition: slide-left
---

# Exercise: useMemo on a filtered list
Refactor Lesson12 (fetching recipes) to include an input box from which to filter fetched recipes

```jsx
  const [searchQuery, setSearchQuery] = useState("");
  ... 
  const filteredRecipes = useMemo(() => {
    console.log("filtering Recipes");
    if (!searchQuery) return recipes;
    return recipes.filter((recipe) =>
      recipe.strMeal.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [searchQuery, recipes]);
  ...
      <hr />
      <input
        type="text"
        placeholder="Search"
        onChange={(e) => setSearchQuery(e.target.value)}
      />
      <hr />

      <ul className={styled.recipes}>
        {filteredRecipes.length > 0 &&
          filteredRecipes.map((recipe) => <li>{recipe.strMeal}</li>)}
      </ul>
    </>
  );
};
```

<!--
- try changing categories state by using React devtools and deleting categories to test if console logs out "filtering recipes"
-->

---
transition: slide-left
---

# Optimization: useCallback

```jsx
  const [count, setCount] = useState(0);
  const [count2, setCount2] = useState(0);

  const increment = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []);

  const prevIncRef = useRef(increment);

  useEffect(() => {
    console.log("Fn changed: ", prevIncRef.current !== increment);
    prevIncRef.current = increment;
  }, [increment]);

  return (
    <>
      <h1>Count: {count}</h1>
      <button onClick={increment}>Increment</button>
      <button onClick={() => setCount2(count2 + 1)}>
        Change different state to trigger re-render
      </button>
    </>
  );
}
```

---
transition: slide-left
---

# Exercise: useCallback on a filtered list

- see app.agencyanalytics.com > dev tools Source tab > Ctrl + P to find TaskModal.tsx
1. Refactor Recipe app that we just did to useCallback for selecting category
  ```jsx
  // to remove all this later: test if useCallback is working via useEffect and useRef
  const prevSelectCategoryFn = useRef(handleSelectCategory);

  useEffect(() => {
    console.log(
      "Function changed: ",
      prevSelectCategoryFn.current !== handleSelectCategory
    );
    prevSelectCategoryFn.current = handleSelectCategory;
  }, [handleSelectCategory]);
  ```
2. Refactor Recipe app to useCallback for filtering search change

---
transition: slide-left
---

# Exercise: Fill-in-the-blank (pg.1)

```jsx
  // 1. Declare state/setter for questions set to the default value of []
  // 2. Declare state/setter for currentIndex set to the default value of 0
  // 3. Declare state/setter for gameOver set to the default value of false
  // 4. Declare const currentQuestion set to the value of the questions array at currentIndex 

  useEffect(() => {
    const fetchQuestions = async () => {
      const res = await fetch(
        "https://opentdb.com/api.php?amount=15&category=18&type=multiple&encode=url3986"
      );
      const data = await res.json();
      // 5. Call the setter function to set questions to be data.results 

    };
    fetchQuestions();
  }, []);

    // 6. Use the appropriate hook to memoize the following value 
    const shuffledAnswers = ______(() => {
    if (!currentQuestion) return [];
    return [...currentQuestion.incorrect_answers, currentQuestion.correct_answer,
    ].sort(() => Math.random() - 0.5);
    // 7. What goes in the dependency array below?
  }, [_______]);
```

---
transition: slide-left
---

# Exercise: Fill-in-the-blank (pg.2)

```jsx
 // 8. Use the appriate hook to memoize the following function
 const handleAnswer = ________(
    (answer: string) => {
      if (!currentQuestion) return;

      if (answer === currentQuestion.correct_answer) {
        setCurrentIndex((prev) => prev + 1);
      } else {
        setGameOver(true);
      }
    },
    // 9. What goes in the dependency array?
    [____________]
  );

  // 10. Use the appriate hook to memoize the following function
  const restartGame = _________(() => {
    setCurrentIndex(0);
    setGameOver(false);
    // 11. What goes in the dependency array?
  }, [________]);

  if (!currentQuestion) return <div>Loading questions...</div>;
```

---
transition: slide-left
---

# Exercise: Fill-in-the-blank (pg.3)

```jsx
 if (gameOver) {
    return (
      <div>
        <h2>Game Over!</h2>
        <!-- 12. Which function reference should we put below to restart the game? -->
        <button onClick={_______}>Play Again</button>
      </div>
    );
  }

  return (
    <div>
      <h2>{decodeURIComponent(currentQuestion.question)}</h2>
      <div style={{ display: "flex", gap: "10px", justifyContent: "center" }}>
        {shuffledAnswers.map((answer, index) => (
          <button key={index} onClick={() => handleAnswer(answer)}>
            {decodeURIComponent(answer)}
          </button>
        ))}
      </div>
    </div>
  );
};
```

---
layout: image-right
transition: slide-left
image: /assets/pubsub.png
backgroundSize: 420px 380px
class: text-left
---

# 10 minute break

🍦 Cool Tips, Trends and Resources:

- ☑️ [CircuitStream needs your feedback](https://forms.gle/SpjofQ82w1boWcqS9)
- 🦆 [Duck AI](https://duck.ai/)
- ⚛️ [React v19 (RSC) Cheat Sheet](https://www.epicreact.dev/react-19-cheatsheet)
- 🛖 [Architectures of modern Front-end Apps](https://blog.meetbrackets.com/architectures-of-modern-front-end-applications-8859dfe6c12e)

<br>
<hr>
<br>

- 🧪 [Enter anonymous lab questions](https://docs.google.com/forms/d/e/1FAIpQLSevvGARdHQikso-uLqFCO481MABKE5HofuSrlzEPMNQ2ZLykw/viewform?usp=dialog)
- ℹ️ [Course feedback survey](https://circuitstream.typeform.com/to/ZoyYk7px#course_id=SoftwareAN&instructor=9514)

---
transition: slide-left
---

# Good Practices

---
transition: slide-left
---

# React Dev Tools > Profiler

---
transition: slide-left
---

# Exercise:

---
transition: slide-left
---

# Homework

- Refactor the weather app we did for our first exercise (see https://codepen.io/codevilla/pen/YPyWWpm) into the following components.  Then separate the css into its respective components that you created.
Feel free to create more components inside `<Forecast>` as you see fit.  FYI - ignore my `this.whatever` or `this.state.whatever` code below since I was using class-based React which you won't be using.
  ```jsx
  return (
      <div>
        <header>
          <Nav city={this.state.currentCity} handleCityChange={this.changeCity} />
        </header>
        <main>
          <TodayWeather
            city={this.state.currentCity}
            handleCoordsChange={this.changeCoords}
          />
          <Forecast lat={this.state.lat} lon={this.state.lon} />
        </main>
      </div>
    );
  ```
- Start working on "Weather Forecasting App" assignment due Aug 17 midnight EST
