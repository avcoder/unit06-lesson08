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

# React: useMemo, UseCallback
Frontend Development: Unit 06 - Lesson 08

- [ ] Optimizations via memo, useMemo, useCallback
- [ ] Good Practices + Profiler
- [ ] useId, useRef

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

# Optimization: React.memo

- Q: Why did H2 re-render if it didn't depend on `count`?
  - A: because React tries to re-render all descendants, regardless.  This is because it's hard to know with 100% certainty, whether H2 depends indirectly on `count`.  
  - React would rather err on the side of caution and re-render all descendants to match UI with state.
- Q: In larger apps, is it possible for one stage change to degrade performance while re-rendering all its children components?
- IF you do notice performance issues, then perhaps using `React.memo()` will help
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

<br>
<hr>
<br>

- 🧪 [Enter anonymous lab questions](https://docs.google.com/forms/d/e/1FAIpQLSevvGARdHQikso-uLqFCO481MABKE5HofuSrlzEPMNQ2ZLykw/viewform?usp=dialog)
- ℹ️ [Course feedback survey](https://circuitstream.typeform.com/to/ZoyYk7px#course_id=SoftwareAN&instructor=9514)

---
transition: slide-left
---

# O

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
