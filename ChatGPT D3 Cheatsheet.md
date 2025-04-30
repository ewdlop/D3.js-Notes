Great. I’ll create a detailed D3.js cheatsheet covering as much as allowed: including selections, data binding, scales, axes, transitions, shapes, layouts, SVG basics, and key differences across major versions.
I’ll let you know when it's ready.

# D3.js Cheatsheet (v7+)

## Selections & Data Binding
- **Selecting Elements:** Use `d3.select()` (selects first match) and `d3.selectAll()` (selects all matches) to target DOM elements. For example:  
  ```js
  const svg = d3.select("#chart")        // First element with id "chart"
    .append("svg")                      // Append an SVG child
    .attr("width", 500).attr("height", 300);
  const circles = svg.selectAll("circle") // Empty selection initially
    .data([30, 50, 80]);
  ```
- **Data Binding:** Call `.data(array[, key])` on a **selection** to bind an array of data to selected elements. The optional *key function* ensures stable binding by identity (minimizing DOM changes on update). If there are more data than existing elements, surplus data form the *enter* selection; if fewer data, extra elements form the *exit* selection ([d3/docs/d3-selection/joining.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/joining.md#:~:text=enter%20%3D%3E%20enter.append%28,black)).
- **Data Access in Callbacks:** Within methods like `.attr` or `.style`, `d` is the bound datum, and optional `i` is the index. Example:  
  ```js
  circles.data([30, 50, 80])
    .enter().append("circle")
      .attr("r", d => d)                // radius = data value
      .attr("cx", (d, i) => 50 + i*80); // space circles horizontally
  ```

## Enter/Update/Exit Pattern
- **General Update Pattern:** Classic approach uses three selections:  
  ```js
  const updateSel = svg.selectAll("circle").data(data, key);
  const enterSel = updateSel.enter().append("circle");
  const exitSel = updateSel.exit();
  
  // Enter + Update
  enterSel.merge(updateSel)    // merge to treat enter+update as one selection
    .attr(...);
  exitSel.remove();
  ```
  - *Enter selection:* new elements for new data (`enterSel.append(...)`).
  - *Update selection:* existing elements bound to data (the `updateSel` itself after `.data`).
  - *Exit selection:* elements with no data (`exitSel.remove()` to delete).
- **`selection.join()` (v6+):** A convenient shortcut that handles enter, update, exit in one call ([d3/docs/d3-selection/joining.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/joining.md#:~:text=svg.selectAll%28,black)). Example:  
  ```js
  svg.selectAll("circle")
    .data(data, key)
    .join(
      enter => enter.append("circle").attr("fill", "green"), // on enter
      update => update.attr("fill", "blue"),                 // on update
      exit => exit.remove()                                  // on exit
    )
    .attr("stroke", "black");
  ```  
  This appends/removes as needed and returns the merged enter+update selection ([d3/docs/d3-selection/joining.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/joining.md#:~:text=enter%20%3D%3E%20enter.append%28,black)). You can also use a **string** for a simple `.append`: `.join("circle")`.

## Scales & Axes
- **Linear Scale:** Maps a continuous numeric domain to a continuous range. Example:  
  ```js
  const x = d3.scaleLinear()                 // create a linear scale
    .domain([0, 100])                        // input domain
    .range([0, 500]);                        // output range (pixels)
  x(50);  // ⇒ 250 (maps 50% of domain to 50% of range)
  x.invert(250);  // ⇒ 50 (invert maps range back to domain)
  ```  
  If only one argument is given, it’s taken as the **range** (domain defaults to [0,1]). Continuous scales support extrapolation beyond the domain unless clamped.
- **Band Scale:** For discrete items (like categories) spaced evenly.  
  ```js
  const xBand = d3.scaleBand()
    .domain(["A", "B", "C"])          // categorical domain
    .range([0, 300])                  // continuous range
    .padding(0.1);                    // 10% padding between bands
  xBand("B");      // position for category "B"
  xBand.bandwidth(); // width of each band
  ```
- **Time Scale:** Like linear scales but for Date objects. E.g., `d3.scaleTime().domain([dateStart, dateEnd]).range([0, width])`.
- **Color Scales:** The `d3.scaleSequential` or `d3.scaleOrdinal` modules provide color mapping. For instance, `d3.scaleOrdinal(d3.schemeCategory10)` maps categories to 10 preset colors. Continuous numeric to color can use `d3.scaleLinear().range(["white", "steelblue"])` (interpolates colors).
- **Axes:** D3 creates axis generators from scales. Example:  
  ```js
  const xAxis = d3.axisBottom(x);  // bottom orientation for x scale
  svg.append("g")
    .attr("transform", `translate(0,${height - margin})`)
    .call(xAxis);
  ```  
  Similar generators: `d3.axisLeft`, `d3.axisRight`, `d3.axisTop`. Customize ticks via `.ticks(count)` or `.tickFormat(format)`.

## SVG Shapes & Layouts
- **Basic SVG Elements:** Create shapes using D3 or set attributes directly:
  - **Circle:** Use `<circle cx="" cy="" r="">`. Example: `svg.append("circle").attr("cx",50).attr("cy",50).attr("r",20)`.
  - **Rectangle:** Use `<rect x="" y="" width="" height="">` plus optional `rx, ry` for rounded corners.
  - **Line/Path:** Use `<line x1="" y1="" x2="" y2="">` or path generators for complex shapes.
- **D3 Shape Generators (d3-shape):** Functions that output SVG path data (`"M..."`) from data. Use with `<path d="...">`:
  - **Line Generator:**  
    ```js
    const line = d3.line()
      .x(d => xScale(d.date))
      .y(d => yScale(d.value)); ([d3/docs/d3-shape/line.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-shape/line.md#:~:text=const%20line%20%3D%20d3,y%28d.Close))
    svg.append("path")
      .datum(dataArray)                 // bind array to single path
      .attr("d", line)                  // calls line generator
      .attr("fill", "none").attr("stroke", "steelblue");
    ```
    Supports `.defined()` to skip null data, `.curve()` for interpolation (e.g., `d3.curveStep`).
  - **Area Generator:** Similar to line, but fills area between two Y values (y0 and y1) ([d3/docs/d3-shape/area.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-shape/area.md#:~:text=const%20area%20%3D%20d3,y%28d.Close)). For example:  
    ```js
    const area = d3.area()
      .x(d => x(d.date))
      .y0(height)                    // baseline (e.g., bottom of chart)
      .y1(d => y(d.value)); ([d3/docs/d3-shape/area.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-shape/area.md#:~:text=const%20area%20%3D%20d3,y%28d.Close))
    svg.append("path").datum(data).attr("d", area);
    ```
    Use `.y0` for baseline and `.y1` for top line (or x0/x1 for horizontal areas).
  - **Pie & Arc Generators:** For pie and donut charts.
    - `d3.pie()` computes start and end angles for an array of values. Example:  
      ```js
      const pie = d3.pie().value(d => d.amount);
      const arcs = pie(data);  // returns array of angle segments
      ``` 
    - `d3.arc()` generates an SVG arc path given angles and radii. Example:  
      ```js
      const arc = d3.arc().innerRadius(50).outerRadius(100);
      svg.selectAll("path.slice")
        .data(arcs)
        .enter().append("path")
          .attr("class", "slice")
          .attr("d", arc);  // each datum has {startAngle, endAngle} from pie()
      ```
    - The arc generator can also be used directly with an object: `arc({startAngle: 0, endAngle: Math.PI, innerRadius:0, outerRadius:50})` returns a semicircular path.
  - **Other Shapes:** D3 provides `d3.symbol()` for point markers (like circles, triangles, etc.), and `d3.stack()` for stacking data (used with area or bar charts). For hierarchical edge connections, `d3.linkHorizontal` or `d3.linkVertical` can create path data for connecting nodes (in tree diagrams, etc.).

## Transitions & Animations
- **Creating a Transition:** Call `.transition()` on a selection to animate changes. For example, to fade out circles:  
  ```js
  d3.selectAll("circle")
    .transition().duration(1000)      // 1 second duration ([d3/docs/d3-transition/timing.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-transition/timing.md#:~:text=transition.duration%28value%29%20%7B))
    .attr("r", 0)                     // animate radius to 0
    .style("opacity", 0);
  ```
  This smoothly interpolates attributes/styles from current to target values.
- **Easing:** Use `.ease(d3.easeCubic)` (default is easeCubic). D3 includes many easing functions (linear, bounce, elastic, etc.). Example: `.ease(d3.easeBounce)` for a bouncing effect.
- **Delay & Duration:** Set `.duration(ms)` ([d3/docs/d3-transition/timing.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-transition/timing.md#:~:text=transition.duration%28value%29%20%7B)) and `.delay(ms)` on transitions to control timing. Both can take a constant or a function `(d, i) => time` to stagger animations per element.
- **Transition Events:** Attach `.on("end", callback)` to run code after transition ends (each element) or `"interrupt"` if canceled.
- **Chaining:** Most selection methods work on transitions too (e.g., `transition.attr`, `transition.style`). However, you must append elements or bind data before starting a transition. To remove elements at the end of a transition, use `transition.remove()`:
  ```js
  circle.exit()
    .transition().duration(500)
    .style("opacity", 0)
    .remove();                        // remove after fade-out
  ```

## Events & Interactivity
- **Adding Event Listeners:** Use `selection.on(eventType, listener)` ([d3/docs/d3-selection/events.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/events.md#:~:text=selection.on%28typenames%2C%20listener%2C%20options%29%20%7B)) to register event callbacks. Example:  
  ```js
  svg.on("click", (event, d) => {       // 'event' is the DOM event
    console.log("SVG clicked!", d);
  });
  ```
  Common events: `"click"`, `"mouseover"`, `"mouseout"`, `"keydown"`, etc.
- **Accessing Event & Element:** In D3 v6+, use the passed `event` (no more `d3.event`). Within a listener, `this` is the current DOM element (or use `event.currentTarget`), and the second argument `d` is the bound data (if any).
- **Pointer Helpers (v6+):** D3 provides `d3.pointer(event, target)` to get [x,y] of an event relative to a given DOM element ([d3/docs/d3-selection/events.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/events.md#:~:text=pointer%28event%2C%20target%29%20%7B)). For example, inside an event listener:
  ```js
  const [x, y] = d3.pointer(event, svg.node());
  ```  
  For multi-touch, use `d3.pointers(event, target)` to get an array of points.
- **Drag Behavior:** D3’s drag module (`d3.drag()`) simplifies making elements draggable. Example:
  ```js
  const dragBehavior = d3.drag()
    .on("start", (event, d) => {/*…*/})
    .on("drag", (event, d) => {
       d.x += event.dx; d.y += event.dy;      // update data
       d3.select(this).attr("cx", d.x).attr("cy", d.y); // update position
    });
  d3.selectAll("circle.node").call(dragBehavior);
  ```
  The drag events provide `event.x, event.y` (new absolute position) and `event.dx, event.dy` (delta movement).  

## Hierarchical Layouts (Tree, Pack, etc.)
- **Hierarchy Data:** Use `d3.hierarchy(rootObject)` to turn nested data (objects with children arrays) into a hierarchy node structure. You can then compute layouts like tree or pack on this hierarchy.
  ```js
  const root = d3.hierarchy(data);        // create root node
  root.sum(d => d.value);                 // (for pack/treemap) sum values in hierarchy
  d3.tree().size([width, height])(root);  // compute tree layout: assigns x, y for nodes
  ```
- **Tree (Node-Link Diagrams):** `d3.tree()` produces a dendrogram (each node has `.x` and `.y`) given a hierarchy. After computing, you can draw links and nodes:
  ```js
  const root = d3.hierarchy(treeData);
  d3.tree().size([width, height])(root);
  // root.descendants() gives array of all nodes; root.links() gives parent-child link pairs
  svg.selectAll("line.link")
      .data(root.links())
      .join("line")
        .attr("class", "link")
        .attr("x1", d => d.source.x).attr("y1", d => d.source.y)
        .attr("x2", d => d.target.x).attr("y2", d => d.target.y);
  svg.selectAll("circle.node")
      .data(root.descendants())
      .join("circle")
        .attr("class", "node")
        .attr("cx", d => d.x).attr("cy", d => d.y)
        .attr("r", 5);
  ```
- **Pack (Circle Packing):** `d3.pack()` computes positions (`node.x, node.y`) and radius (`node.r`) for each node in a hierarchy. Usage:
  ```js
  const root = d3.hierarchy(data).sum(d => d.value);
  d3.pack().size([width, height]).padding(3)(root);  // compute pack layout
  svg.selectAll("circle")
    .data(root.descendants())
    .join("circle")
      .attr("cx", d => d.x).attr("cy", d => d.y).attr("r", d => d.r);
  ```
  Remember to call `root.sum(...)` (and optionally `root.sort(...)`) before packing.
- **Other Hierarchical Layouts:**  
  - **Cluster:** `d3.cluster()` similar to tree but compacts leaves (like a radial layout when combined with polar coords).
  - **Treemap:** `d3.treemap()` computes rectangular areas for each node (node.x0, x1, y0, y1 for corners). Use `root.sum` for values and assign a `.size([...])`.
  - **Partition (Sunburst):** `d3.partition()` computes angles and radial coordinates for sunburst charts.

## Utilities (Arrays, Statistics, etc.)
- **Arrays & Statistics (d3-array):** D3 offers many helpers for array data:
  - `d3.extent(array, accessor)` – returns [min, max] of values.
  - `d3.max`, `d3.min`, `d3.mean`, `d3.median` – summary stats.
  - `d3.range(start, stop, step)` – generates an array of numbers (e.g., `d3.range(0, 5, 1)` → [0,1,2,3,4]).
  - `d3.bin()` – histogram bin generator (to bucket numeric data).
  - `d3.shuffle(array)` – randomize array order.
- **Grouping Data:** In D3 v6+, use `d3.group(data, ...keys)` to group data into nested Maps ([d3/docs/d3-array/group.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-array/group.md#:~:text=const%20species%20%3D%20d3,species)). Example:
  ```js
  const nested = d3.group(penguins, d => d.species, d => d.sex);
  // nested is a Map: species -> sex -> [objects]
  ```
  For arrays of entries instead of Maps, use `d3.groups` ([d3/docs/d3-array/group.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-array/group.md#:~:text=Equivalent%20to%20group%2C%20but%20returns,first%20instance%20of%20each%20key)).  
  **Rollups:** `d3.rollup(data, reduceFn, ...keys)` computes aggregate values. Example:
  ```js
  const counts = d3.rollup(penguins, v => v.length, d => d.species);
  // counts is a Map: species -> count ([d3/docs/d3-array/group.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-array/group.md#:~:text=const%20speciesCount%20%3D%20d3,d.species))
  ```
  For array-of-entries output, use `d3.rollups`.
- **Nesting (Deprecated):** `d3.nest` was replaced by `d3.group/rollup` in v6 for a more modern API (returns Map, works well with iterables).
- **d3.scaleChromatic:** Preset color scales like `d3.interpolateBlues` or category palettes in `d3.schemeCategory10`.
- **d3.format:** Number formatting (e.g., `d3.format(".2f")(Math.PI)` → "3.14").
- **d3.timeFormat:** Date/time formatting (e.g., `d3.timeFormat("%Y-%m-%d")(new Date())`).
- **d3.interpolate:** Interpolation functions (e.g., `d3.interpolateRgb("red", "blue")(0.5)` gives a mid color).

## Best Practices & Patterns
- **Separation of Concerns:** Use D3 for *data-to-DOM binding and transitions*, but structure code to keep data logic separate from rendering. Embrace the join pattern or use frameworks in combination with D3 for complex state management.
- **UPDATE Pattern:** Always consider how your code handles enter/update/exit to keep visualizations in sync with data. Use keys when data has an inherent ID.
- **Performance:** Minimize DOM elements when possible (e.g., use canvas for very large sets). Use `requestAnimationFrame` for smooth animations when not using D3 transitions.
- **Styling:** Prefer CSS for static styles and D3 for dynamic ones (e.g., set classes with `.attr("class", ...)` then style via CSS).
- **Module Approach:** D3 v4+ is modular. You can import only what you need (e.g., `import {select, scaleLinear} from "d3";`). The official docs are organized by modules (d3-selection, d3-scale, etc.), which are listed on [D3’s API Reference][d3-api].

## Version Notes (v5 vs v6 vs v7)
- **Data Join:** v6 introduced `selection.join()` to simplify the enter/update/exit pattern.
- **Events:** As of v6, `d3.event` (from v5 and earlier) is removed. Use the event passed to listeners (as shown above) or `d3.pointer` utilities ([d3/docs/d3-selection/events.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/events.md#:~:text=pointer%28event%2C%20target%29%20%7B)).
- **Promises & Fetch:** v5 replaced d3-request with d3-fetch, using modern Promises (often with async/await) for loading data. For example, `d3.csv(url).then(data => ...)` or `const data = await d3.csv(url)` ([d3/docs/d3-fetch.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-fetch.md#:~:text=const%20data%20%3D%20await%20d3.csv%28%22hello,%E2%80%A6)).
- **Deprecated `d3.nest`:** v6 removed d3-collection and `d3.nest`. Use `d3.group` / `d3.rollup` for grouping ([d3/docs/d3-array/group.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-array/group.md#:~:text=Equivalent%20to%20group%2C%20but%20returns,first%20instance%20of%20each%20key)).
- **Selection Order:** v5+ preserves document order in merges by default (no more `.order()` needed in typical use).
- **New in v7:** Additional color schemes and interpolation tweaks, improved curve (spline) calculation, performance improvements in layout algorithms, etc. *(Check official release notes for details.)*

## Further Reading
- **Official Documentation:** Refer to the [official D3 API reference][d3-api] for detailed docs and examples (each module has docs on D3’s site or Observable notebooks). Key references include Selections, Data Joins ([d3/docs/d3-selection/joining.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/joining.md#:~:text=svg.selectAll%28,exit.remove%28%29)), Scales, Transitions, and Layouts.
- **D3 Bostock’s Blocks & Observable:** Mike Bostock’s examples on Observable (e.g., the Notebooks linked in the docs) are great for learning specific techniques ([d3/docs/d3-selection/joining.md at main · d3/d3 · GitHub](https://github.com/d3/d3/blob/main/docs/d3-selection/joining.md#:~:text=.attr%28)).
- **Community:** The D3 community and tutorials (e.g., blocks.org examples, Stack Overflow) provide practical snippets and patterns for common tasks.

[d3-api]: https://github.com/d3/d3/blob/main/docs/README.md "D3 Official API Documentation"