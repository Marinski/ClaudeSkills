---
name: d3js-visualization
description: "Professional data visualization creation using D3.js with support for interactive charts, custom visualizations, animations, and responsive design. Use for: (1) Creating custom interactive charts, (2) Building dashboards, (3) Network/graph visualizations, (4) Geographic data mapping, (5) Time series analysis, (6) Real-time data visualization, (7) Complex multi-dimensional data displays"
---

# D3.js Data Visualization Skill

## Table of Contents
1. [What is D3.js](#what-is-d3js)
2. [D3.js Fundamentals](#d3js-fundamentals)
3. [Chart Types and Use Cases](#chart-types-and-use-cases)
4. [Core D3 Concepts](#core-d3-concepts)
5. [Detailed Workflows](#detailed-workflows)
6. [Best Practices](#best-practices)
7. [Data Transformation](#data-transformation)
8. [Advanced Techniques](#advanced-techniques)
9. [Common Pitfalls](#common-pitfalls)
10. [Integration Patterns](#integration-patterns)

---

## What is D3.js

D3.js (Data-Driven Documents) is a JavaScript library for producing dynamic, interactive data visualizations in web browsers. It uses HTML, SVG, and CSS standards to bind data to the DOM and apply data-driven transformations.

### When to Use D3.js

**Choose D3.js when you need:**
- Custom, unique visualizations not available in chart libraries
- Fine-grained control over every visual element
- Complex interactions and animations
- Data-driven DOM manipulation beyond just charts
- Performance with large datasets (when using Canvas)
- Web standards-based visualizations

**Consider alternatives when:**
- Simple standard charts are sufficient (use Chart.js, Plotly)
- Quick prototyping is priority (use Observable, Vega-Lite)
- Static charts for print/reports (use matplotlib, ggplot2)
- 3D visualizations (use Three.js, WebGL libraries)

### D3.js vs Other Libraries

| Library | Best For | Learning Curve | Customization |
|---------|----------|----------------|---------------|
| D3.js | Custom visualizations | Steep | Complete |
| Chart.js | Standard charts | Easy | Limited |
| Plotly | Scientific plots | Medium | Good |
| Highcharts | Business dashboards | Easy | Good |
| Three.js | 3D graphics | Steep | Complete |

---

## D3.js Fundamentals

### SVG Basics for D3

D3 primarily works with SVG (Scalable Vector Graphics), an XML-based markup language for describing 2D graphics.

#### Essential SVG Elements

```html
<!-- Rectangle -->
<rect x="10" y="10" width="100" height="50" fill="blue" />

<!-- Circle -->
<circle cx="50" cy="50" r="30" fill="red" />

<!-- Line -->
<line x1="0" y1="0" x2="100" y2="100" stroke="black" stroke-width="2" />

<!-- Path (complex shapes) -->
<path d="M 10 10 L 50 50 L 10 90 Z" fill="green" />

<!-- Text -->
<text x="50" y="50" font-size="14" fill="black">Hello</text>

<!-- Group (for transformations) -->
<g transform="translate(50, 50) rotate(45)">
  <!-- elements here -->
</g>
```

#### SVG Coordinate System

- Origin (0,0) is at **top-left** corner
- X increases to the right
- Y increases **downward** (different from Cartesian)
- Use `transform="translate(x, y)"` to reposition

### Data Binding Concepts

D3's core power comes from binding data to DOM elements.

#### Basic Data Binding

```javascript
// Bind array to paragraphs
const data = [10, 20, 30, 40];

d3.select("body")
  .selectAll("p")
  .data(data)
  .enter()
  .append("p")
  .text(d => `Value: ${d}`);
```

#### The Enter-Update-Exit Pattern

This is D3's fundamental data join pattern:

```javascript
// Initial data
const data = [1, 2, 3, 4, 5];

// Create selection
const circles = svg.selectAll("circle")
  .data(data);

// ENTER: Create new elements for new data
circles.enter()
  .append("circle")
  .attr("r", 5)
  .merge(circles) // Merge with existing
  .attr("cx", (d, i) => i * 50)
  .attr("cy", d => d * 10);

// EXIT: Remove elements without data
circles.exit().remove();
```

**Modern D3 v6+ Join Pattern:**

```javascript
svg.selectAll("circle")
  .data(data)
  .join(
    enter => enter.append("circle")
      .attr("r", 0)
      .call(enter => enter.transition().attr("r", 5)),
    update => update.attr("fill", "blue"),
    exit => exit.call(exit => exit.transition().attr("r", 0).remove())
  )
  .attr("cx", (d, i) => i * 50)
  .attr("cy", d => d * 10);
```

### Selections and Manipulation

#### Selection Methods

```javascript
// Select single element (first match)
d3.select("body")
d3.select("#myId")
d3.select(".myClass")

// Select all elements (all matches)
d3.selectAll("p")
d3.selectAll(".item")

// Select within selection
const container = d3.select("#container");
container.selectAll(".item")
```

#### Manipulation Methods

```javascript
// Set attributes
selection.attr("class", "highlight")
selection.attr("cx", 50)
selection.attr("cy", d => d.value * 10) // Function of data

// Set styles
selection.style("color", "red")
selection.style("font-size", "14px")

// Set properties
selection.property("value", "text")
selection.property("checked", true)

// Set text/HTML
selection.text("Hello")
selection.html("<strong>Bold</strong>")

// Append/Insert/Remove
selection.append("div")
selection.insert("p", ":first-child")
selection.remove()

// Class manipulation
selection.classed("active", true)
selection.classed("highlight", d => d.value > 100)
```

### Scales and Axes

Scales map data values (domain) to visual values (range).

#### Scale Types

**Continuous Scales:**

```javascript
// Linear scale
const xScale = d3.scaleLinear()
  .domain([0, 100])        // Data range
  .range([0, 500]);        // Pixel range
xScale(50); // Returns 250

// Log scale (for exponential data)
const logScale = d3.scaleLog()
  .domain([1, 1000])
  .range([0, 500]);

// Power scale
const powScale = d3.scalePow()
  .exponent(2)
  .domain([0, 100])
  .range([0, 500]);

// Square root scale (common for area)
const sqrtScale = d3.scaleSqrt()
  .domain([0, 100])
  .range([0, 500]);

// Time scale
const timeScale = d3.scaleTime()
  .domain([new Date(2020, 0, 1), new Date(2021, 0, 1)])
  .range([0, 500]);
```

**Discrete Scales:**

```javascript
// Ordinal scale (categorical)
const colorScale = d3.scaleOrdinal()
  .domain(["A", "B", "C"])
  .range(["red", "green", "blue"]);

// Band scale (for bar charts)
const xScale = d3.scaleBand()
  .domain(["Mon", "Tue", "Wed", "Thu", "Fri"])
  .range([0, 500])
  .padding(0.1);

xScale("Mon"); // Returns x position
xScale.bandwidth(); // Returns bar width

// Point scale (for scatter plots)
const pointScale = d3.scalePoint()
  .domain(["A", "B", "C"])
  .range([0, 500])
  .padding(0.5);
```

**Color Scales:**

```javascript
// Sequential (for continuous data)
const colorScale = d3.scaleSequential()
  .domain([0, 100])
  .interpolator(d3.interpolateBlues);

// Diverging (for data with meaningful center)
const divergingScale = d3.scaleDiverging()
  .domain([0, 50, 100])
  .interpolator(d3.interpolateRdYlGn);

// Quantize (discrete bins)
const quantizeScale = d3.scaleQuantize()
  .domain([0, 100])
  .range(["low", "medium", "high"]);

// Threshold (custom breakpoints)
const thresholdScale = d3.scaleThreshold()
  .domain([10, 50, 90])
  .range(["#eee", "#ccc", "#999", "#666"]);
```

#### Creating Axes

```javascript
// Define scales
const xScale = d3.scaleLinear()
  .domain([0, 100])
  .range([0, 500]);

const yScale = d3.scaleLinear()
  .domain([0, 50])
  .range([400, 0]); // Inverted for bottom-up

// Create axis generators
const xAxis = d3.axisBottom(xScale)
  .ticks(10)
  .tickFormat(d => `$${d}`);

const yAxis = d3.axisLeft(yScale)
  .ticks(5);

// Append axes to SVG
svg.append("g")
  .attr("class", "x-axis")
  .attr("transform", `translate(0, ${height})`)
  .call(xAxis);

svg.append("g")
  .attr("class", "y-axis")
  .call(yAxis);
```

**Axis Customization:**

```javascript
const axis = d3.axisBottom(xScale)
  .ticks(10)                    // Number of ticks
  .tickSize(6)                  // Tick length
  .tickPadding(3)               // Space between tick and label
  .tickFormat(d3.format(".2f")) // Format numbers
  .tickValues([0, 25, 50, 75, 100]); // Specific tick values
```

### Transitions and Animations

Transitions smoothly interpolate between states.

#### Basic Transitions

```javascript
// Simple transition
d3.select("circle")
  .transition()
  .duration(1000)        // 1 second
  .attr("r", 50)
  .attr("fill", "red");

// With easing
selection
  .transition()
  .duration(500)
  .ease(d3.easeBounceOut)
  .attr("cx", 100);
```

#### Easing Functions

```javascript
// Common easing functions
d3.easeLinear        // Constant speed
d3.easeCubic         // Slow-fast-slow
d3.easeBounce        // Bounce at end
d3.easeElastic       // Elastic oscillation
d3.easeBack          // Overshoot and return
d3.easeSin           // Sinusoidal
d3.easeExp           // Exponential
d3.easeCircle        // Circular

// In/Out/InOut variants
d3.easeCubicIn       // Slow start
d3.easeCubicOut      // Slow end
d3.easeCubicInOut    // Slow start and end
```

#### Chaining Transitions

```javascript
selection
  .transition()
  .duration(500)
  .attr("r", 50)
  .transition()        // Chain next transition
  .duration(500)
  .attr("fill", "red")
  .transition()
  .duration(500)
  .attr("cx", 100);
```

#### Transition Events

```javascript
selection
  .transition()
  .duration(1000)
  .attr("r", 50)
  .on("start", function() {
    console.log("Transition started");
  })
  .on("end", function() {
    console.log("Transition ended");
  })
  .on("interrupt", function() {
    console.log("Transition interrupted");
  });
```

#### Staggered Transitions

```javascript
// Delay each element
circles
  .transition()
  .duration(500)
  .delay((d, i) => i * 100)  // 100ms delay between each
  .attr("r", 10);
```

### Event Handling and Interactivity

#### Mouse Events

```javascript
selection
  .on("click", function(event, d) {
    console.log("Clicked:", d);
    console.log("Element:", this);
    console.log("Event:", event);
  })
  .on("mouseover", function(event, d) {
    d3.select(this)
      .transition()
      .attr("r", 15);
  })
  .on("mouseout", function(event, d) {
    d3.select(this)
      .transition()
      .attr("r", 10);
  })
  .on("mousemove", function(event, d) {
    const [x, y] = d3.pointer(event);
    console.log(`Mouse at: ${x}, ${y}`);
  });
```

#### Drag Behavior

```javascript
const drag = d3.drag()
  .on("start", function(event, d) {
    d3.select(this).raise().attr("stroke", "black");
  })
  .on("drag", function(event, d) {
    d3.select(this)
      .attr("cx", event.x)
      .attr("cy", event.y);
  })
  .on("end", function(event, d) {
    d3.select(this).attr("stroke", null);
  });

circles.call(drag);
```

#### Zoom Behavior

```javascript
const zoom = d3.zoom()
  .scaleExtent([0.5, 10])  // Min/max zoom
  .on("zoom", function(event) {
    svg.attr("transform", event.transform);
  });

svg.call(zoom);

// Programmatic zoom
svg.transition()
  .duration(750)
  .call(zoom.scaleTo, 2); // Zoom to 2x
```

#### Brush Selection

```javascript
const brush = d3.brush()
  .extent([[0, 0], [width, height]])
  .on("start brush end", function(event) {
    if (event.selection) {
      const [[x0, y0], [x1, y1]] = event.selection;
      console.log(`Selected area: ${x0},${y0} to ${x1},${y1}`);
    }
  });

svg.append("g")
  .attr("class", "brush")
  .call(brush);
```

---

## Chart Types and Use Cases

### Line Charts
**Use for:** Time series, trends, continuous data

**Best practices:**
- Use for showing change over time
- Good for 1-5 lines; more becomes cluttered
- Always include axis labels and legend
- Consider using area charts for cumulative data

**When to avoid:**
- Discrete categories (use bar chart)
- Comparing many series (use small multiples)
- Part-to-whole relationships (use pie/treemap)

### Bar Charts
**Use for:** Comparisons, distributions, categorical data

**Variants:**
- **Vertical bars:** Standard comparisons
- **Horizontal bars:** Long category names, rankings
- **Grouped bars:** Comparing subcategories
- **Stacked bars:** Part-to-whole with categories
- **Normalized stacked:** Percentage composition

**Best practices:**
- Always start axis at zero (or use break indicator)
- Sort bars logically (by value, alphabetically, or custom)
- Use consistent color scheme
- Limit to 10-15 categories

### Scatter Plots
**Use for:** Correlations, distributions, outliers, clustering

**Best practices:**
- Use when showing relationship between two variables
- Color by third dimension or category
- Size by magnitude (bubble chart)
- Add trend line if correlation exists
- Use transparency for overlapping points

**Enhancements:**
- Add brush for selection
- Link to detail view
- Annotate outliers
- Include confidence intervals

### Pie/Donut Charts
**Use for:** Part-to-whole relationships (with caution)

**Best practices:**
- Limit to 5-7 slices maximum
- Sort slices by size
- Start largest slice at 12 o'clock
- Use donut for displaying total in center
- Consider bar chart as alternative

**When to avoid:**
- Comparing similar values (human eye poor at angle comparison)
- More than 7 categories
- Showing change over time

### Network Graphs
**Use for:** Relationships, hierarchies, connections

**Layouts:**
- **Force-directed:** General networks
- **Tree:** Hierarchical data
- **Sankey:** Flow and magnitude
- **Chord:** Relationships between entities

**Best practices:**
- Limit nodes for readability (cluster if needed)
- Use color for node types
- Size nodes by importance/degree
- Weight edges by connection strength
- Add interactivity (hover, drag, click)

---

## Core D3 Concepts

### Paths and Shapes

#### Line Generator

```javascript
const line = d3.line()
  .x(d => xScale(d.date))
  .y(d => yScale(d.value))
  .curve(d3.curveMonotoneX); // Smooth curve

const pathData = line(data);
svg.append("path")
  .attr("d", pathData)
  .attr("fill", "none")
  .attr("stroke", "steelblue");
```

**Curve Types:**

```javascript
d3.curveLinear          // Straight lines (default)
d3.curveBasis           // B-spline
d3.curveCardinal        // Cardinal spline
d3.curveCatmullRom      // Catmull-Rom spline
d3.curveMonotoneX       // Monotone cubic (good for time series)
d3.curveStep            // Step function
d3.curveStepBefore      // Step before
d3.curveStepAfter       // Step after
```

#### Area Generator

```javascript
const area = d3.area()
  .x(d => xScale(d.date))
  .y0(height)              // Baseline
  .y1(d => yScale(d.value)) // Top line
  .curve(d3.curveMonotoneX);

svg.append("path")
  .attr("d", area(data))
  .attr("fill", "steelblue")
  .attr("opacity", 0.3);
```

#### Arc Generator

```javascript
const arc = d3.arc()
  .innerRadius(0)           // 0 for pie, >0 for donut
  .outerRadius(100)
  .padAngle(0.02)           // Gap between slices
  .cornerRadius(3);         // Rounded corners

// Use with pie layout
const pie = d3.pie()
  .value(d => d.value)
  .sort(null);              // Don't sort

const arcs = svg.selectAll(".arc")
  .data(pie(data))
  .enter()
  .append("g")
  .attr("class", "arc");

arcs.append("path")
  .attr("d", arc)
  .attr("fill", (d, i) => colorScale(i));
```

### Layouts

#### Force Simulation

```javascript
const simulation = d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links)
    .id(d => d.id)
    .distance(50))
  .force("charge", d3.forceManyBody()
    .strength(-100))
  .force("center", d3.forceCenter(width / 2, height / 2))
  .force("collision", d3.forceCollide()
    .radius(20))
  .on("tick", ticked);

function ticked() {
  // Update positions on each simulation step
  link
    .attr("x1", d => d.source.x)
    .attr("y1", d => d.source.y)
    .attr("x2", d => d.target.x)
    .attr("y2", d => d.target.y);

  node
    .attr("cx", d => d.x)
    .attr("cy", d => d.y);
}
```

---

## Detailed Workflows

### Setting Up a D3 Project

#### Option 1: CDN (Quick Start)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>D3 Visualization</title>
  <style>
    body { margin: 0; font-family: sans-serif; }
    svg { display: block; }
  </style>
</head>
<body>
  <div id="chart"></div>

  <!-- D3.js v7 -->
  <script src="https://d3js.org/d3.v7.min.js"></script>
  <script>
    // Your code here
  </script>
</body>
</html>
```

#### Option 2: NPM (Production)

```bash
npm install d3

# Or install specific modules
npm install d3-selection d3-scale d3-axis d3-shape
```

```javascript
// Import all of D3
import * as d3 from "d3";

// Or import specific modules
import { select, selectAll } from "d3-selection";
import { scaleLinear, scaleTime } from "d3-scale";
import { axisBottom, axisLeft } from "d3-axis";
import { line, area } from "d3-shape";
```

### Loading and Transforming Data

#### Loading Data

```javascript
// CSV
d3.csv("data.csv").then(data => {
  console.log(data);
  // Auto-parsed into array of objects
});

// JSON
d3.json("data.json").then(data => {
  console.log(data);
});

// Multiple files
Promise.all([
  d3.csv("data1.csv"),
  d3.json("data2.json")
]).then(([csv, json]) => {
  console.log(csv, json);
});
```

#### Parsing and Type Conversion

```javascript
d3.csv("data.csv", function(d) {
  return {
    date: new Date(d.date),
    value: +d.value,           // Convert to number
    category: d.category,       // Keep as string
    active: d.active === "true" // Convert to boolean
  };
}).then(data => {
  console.log(data);
});
```

#### Data Transformation

```javascript
// Filtering
const filtered = data.filter(d => d.value > 50);

// Mapping
const values = data.map(d => d.value);

// Sorting
const sorted = data.sort((a, b) => b.value - a.value);

// Grouping (D3 helper)
const grouped = d3.group(data, d => d.category);
// Returns Map: category -> array of objects

// Rollup (aggregate)
const summed = d3.rollup(
  data,
  v => d3.sum(v, d => d.value),  // Aggregation function
  d => d.category                 // Grouping key
);
// Returns Map: category -> sum

// Extent (min and max)
const [min, max] = d3.extent(data, d => d.value);

// Other statistics
d3.min(data, d => d.value)
d3.max(data, d => d.value)
d3.sum(data, d => d.value)
d3.mean(data, d => d.value)
d3.median(data, d => d.value)
```

### Creating a Basic Chart

#### Complete Workflow

```javascript
// 1. Set up dimensions and margins
const margin = {top: 20, right: 30, bottom: 40, left: 50};
const width = 800 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

// 2. Create SVG
const svg = d3.select("#chart")
  .append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
  .append("g")
  .attr("transform", `translate(${margin.left},${margin.top})`);

// 3. Load data
d3.csv("data.csv", d => ({
  date: new Date(d.date),
  value: +d.value
})).then(data => {

  // 4. Create scales
  const xScale = d3.scaleTime()
    .domain(d3.extent(data, d => d.date))
    .range([0, width]);

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .nice()
    .range([height, 0]);

  // 5. Create axes
  const xAxis = d3.axisBottom(xScale);
  const yAxis = d3.axisLeft(yScale);

  svg.append("g")
    .attr("class", "x-axis")
    .attr("transform", `translate(0,${height})`)
    .call(xAxis);

  svg.append("g")
    .attr("class", "y-axis")
    .call(yAxis);

  // 6. Create line generator
  const line = d3.line()
    .x(d => xScale(d.date))
    .y(d => yScale(d.value))
    .curve(d3.curveMonotoneX);

  // 7. Draw line
  svg.append("path")
    .datum(data)
    .attr("class", "line")
    .attr("d", line)
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 2);

  // 8. Add labels
  svg.append("text")
    .attr("x", width / 2)
    .attr("y", height + margin.bottom - 5)
    .attr("text-anchor", "middle")
    .text("Date");

  svg.append("text")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", -margin.left + 15)
    .attr("text-anchor", "middle")
    .text("Value");
});
```

### Adding Interactivity

#### Tooltips

```javascript
// Create tooltip div
const tooltip = d3.select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("visibility", "hidden")
  .style("background", "white")
  .style("border", "1px solid #ddd")
  .style("padding", "10px")
  .style("border-radius", "4px")
  .style("pointer-events", "none");

// Add to elements
circles
  .on("mouseover", function(event, d) {
    tooltip
      .style("visibility", "visible")
      .html(`<strong>${d.name}</strong><br/>Value: ${d.value}`);
  })
  .on("mousemove", function(event) {
    tooltip
      .style("top", (event.pageY - 10) + "px")
      .style("left", (event.pageX + 10) + "px");
  })
  .on("mouseout", function() {
    tooltip.style("visibility", "hidden");
  });
```

#### Hover Effects

```javascript
circles
  .on("mouseover", function(event, d) {
    d3.select(this)
      .transition()
      .duration(200)
      .attr("r", 8)
      .attr("fill", "orange");
  })
  .on("mouseout", function(event, d) {
    d3.select(this)
      .transition()
      .duration(200)
      .attr("r", 5)
      .attr("fill", "steelblue");
  });
```

### Implementing Responsive Design

```javascript
function createChart() {
  // Get container width
  const container = d3.select("#chart");
  const containerWidth = container.node().getBoundingClientRect().width;

  // Calculate dimensions
  const margin = {top: 20, right: 30, bottom: 40, left: 50};
  const width = containerWidth - margin.left - margin.right;
  const height = Math.min(width * 0.6, 500); // Aspect ratio

  // Clear previous chart
  container.selectAll("*").remove();

  // Create SVG
  const svg = container
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // ... rest of chart code
}

// Initial render
createChart();

// Re-render on resize
window.addEventListener("resize", createChart);

// Or with debouncing
let resizeTimer;
window.addEventListener("resize", () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(createChart, 250);
});
```

---

## Best Practices

### Chart Selection Guidance

**Decision Tree:**

1. **Showing change over time?**
   - Continuous data → Line chart
   - Discrete periods → Bar chart
   - Multiple metrics → Small multiples or area chart

2. **Comparing categories?**
   - Few categories (<10) → Bar chart
   - Many categories → Dot plot or horizontal bar
   - Multiple groups → Grouped or small multiples
   - Part-to-whole → Stacked bar or treemap

3. **Showing relationships?**
   - Two variables → Scatter plot
   - Three variables → Bubble chart (color/size)
   - Network data → Force-directed graph
   - Hierarchical → Tree diagram

4. **Showing distribution?**
   - Single variable → Histogram
   - Multiple groups → Box plot or violin plot
   - Two variables → Heatmap or 2D histogram

### Responsive Design Patterns

#### Container Query

```javascript
function createResponsiveChart() {
  const container = document.getElementById("chart");
  const width = container.clientWidth;

  // Adjust based on width
  const margin = width < 600
    ? {top: 10, right: 10, bottom: 30, left: 40}
    : {top: 20, right: 30, bottom: 40, left: 60};

  const height = width < 600 ? 300 : 500;

  // Adjust tick counts
  const xTicks = width < 600 ? 5 : 10;
  const yTicks = width < 600 ? 5 : 10;

  // ... rest of chart
}
```

#### ViewBox Scaling

```javascript
// Use fixed internal dimensions
const width = 800;
const height = 600;

const svg = d3.select("#chart")
  .append("svg")
  .attr("viewBox", `0 0 ${width} ${height}`)
  .attr("preserveAspectRatio", "xMidYMid meet")
  .style("width", "100%")
  .style("height", "auto");
```

### Performance Optimization

#### For Large Datasets (>1000 points)

**1. Use Canvas instead of SVG:**

```javascript
const canvas = d3.select("#chart")
  .append("canvas")
  .attr("width", width)
  .attr("height", height);

const context = canvas.node().getContext("2d");

// Draw with canvas API
data.forEach(d => {
  context.beginPath();
  context.arc(xScale(d.x), yScale(d.y), 3, 0, 2 * Math.PI);
  context.fillStyle = "steelblue";
  context.fill();
});
```

**2. Aggregate data:**

```javascript
// Bin data for histograms
const bins = d3.bin()
  .domain(xScale.domain())
  .thresholds(20)(data);

// Sample data
const sampledData = data.filter((d, i) => i % 10 === 0);
```

### Color Palette Selection

#### Sequential (For Continuous Data)

```javascript
// Single hue (0 to high)
d3.interpolateBlues
d3.interpolateGreens
d3.interpolateReds

// Multi-hue
d3.interpolateViridis  // Purple to yellow (colorblind-safe)
d3.interpolatePlasma   // Purple to orange
d3.interpolateInferno  // Black to yellow
d3.interpolateTurbo    // Blue to red (high contrast)
```

#### Categorical (For Discrete Groups)

```javascript
// Qualitative schemes
d3.schemeCategory10   // 10 colors
d3.schemeAccent       // 8 colors
d3.schemeSet1         // 9 colors
d3.schemeSet2         // 8 colors (pastels)
d3.schemeSet3         // 12 colors

const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
```

### Accessibility

#### ARIA Labels

```javascript
svg
  .attr("role", "img")
  .attr("aria-label", "Line chart showing sales over time");

// Add description
svg.append("desc")
  .text("A line chart displaying monthly sales from January to December 2023. Sales increased from $100k to $180k over the year.");
```

#### Keyboard Navigation

```javascript
circles
  .attr("tabindex", 0)
  .attr("role", "button")
  .attr("aria-label", d => `Data point: ${d.name}, value: ${d.value}`)
  .on("keydown", function(event, d) {
    if (event.key === "Enter" || event.key === " ") {
      // Trigger interaction
      handleClick(d);
    }
  });
```

---

## Data Transformation

### Data Aggregation

#### Grouping

```javascript
// Group by single key
const byCategory = d3.group(data, d => d.category);
// Map { "A" => [{...}, {...}], "B" => [{...}] }

// Group by multiple keys
const byYearAndCategory = d3.group(
  data,
  d => d.year,
  d => d.category
);
// Map { 2020 => Map { "A" => [...], "B" => [...] }, ... }

// Index (returns single value per key)
const byId = d3.index(data, d => d.id);
// Map { "id1" => {...}, "id2" => {...} }
```

#### Rollup (Aggregation)

```javascript
// Sum values by category
const sumByCategory = d3.rollup(
  data,
  v => d3.sum(v, d => d.value),
  d => d.category
);
// Map { "A" => 150, "B" => 200 }

// Multiple aggregations
const statsByCategory = d3.rollup(
  data,
  v => ({
    count: v.length,
    sum: d3.sum(v, d => d.value),
    mean: d3.mean(v, d => d.value),
    min: d3.min(v, d => d.value),
    max: d3.max(v, d => d.value)
  }),
  d => d.category
);
```

### Date/Time Handling

#### Parsing Dates

```javascript
// Built-in formats
const parser = d3.timeParse("%Y-%m-%d");
const date = parser("2023-01-15");

// Format specifiers
// %Y - 4-digit year (2023)
// %y - 2-digit year (23)
// %m - month (01-12)
// %d - day (01-31)
// %H - hour 24h (00-23)
// %I - hour 12h (01-12)
// %M - minute (00-59)
// %S - second (00-59)
// %p - AM/PM

const parser2 = d3.timeParse("%m/%d/%Y %I:%M %p");
const date2 = parser2("01/15/2023 02:30 PM");
```

#### Formatting Dates

```javascript
const formatter = d3.timeFormat("%b %d, %Y");
formatter(new Date(2023, 0, 15)); // "Jan 15, 2023"

// Common formats
d3.timeFormat("%Y-%m-%d")(date)        // "2023-01-15"
d3.timeFormat("%B %d, %Y")(date)       // "January 15, 2023"
d3.timeFormat("%b %d")(date)           // "Jan 15"
d3.timeFormat("%m/%d/%Y")(date)        // "01/15/2023"
d3.timeFormat("%I:%M %p")(date)        // "02:30 PM"
```

---

## Advanced Techniques

### Reusable Chart Pattern

```javascript
function lineChart() {
  // Configuration variables
  let width = 800;
  let height = 400;
  let margin = {top: 20, right: 30, bottom: 40, left: 50};
  let xValue = d => d.date;
  let yValue = d => d.value;
  let lineColor = "steelblue";

  function chart(selection) {
    selection.each(function(data) {
      // Calculate inner dimensions
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;

      // Update scales
      const xScale = d3.scaleTime()
        .domain(d3.extent(data, xValue))
        .range([0, innerWidth]);

      const yScale = d3.scaleLinear()
        .domain([0, d3.max(data, yValue)])
        .range([innerHeight, 0]);

      // Select or create SVG
      const svg = d3.select(this)
        .selectAll("svg")
        .data([null]);

      const svgEnter = svg.enter()
        .append("svg");

      const g = svgEnter.append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      // Axes
      g.append("g").attr("class", "x-axis")
        .attr("transform", `translate(0,${innerHeight})`);
      g.append("g").attr("class", "y-axis");
      g.append("path").attr("class", "line");

      // Update
      const svgUpdate = svg.merge(svgEnter)
        .attr("width", width)
        .attr("height", height);

      const gUpdate = svgUpdate.select("g");

      gUpdate.select(".x-axis").call(d3.axisBottom(xScale));
      gUpdate.select(".y-axis").call(d3.axisLeft(yScale));

      const line = d3.line()
        .x(d => xScale(xValue(d)))
        .y(d => yScale(yValue(d)))
        .curve(d3.curveMonotoneX);

      gUpdate.select(".line")
        .datum(data)
        .attr("d", line)
        .attr("fill", "none")
        .attr("stroke", lineColor)
        .attr("stroke-width", 2);
    });
  }

  // Getter/setter methods
  chart.width = function(value) {
    if (!arguments.length) return width;
    width = value;
    return chart;
  };

  chart.height = function(value) {
    if (!arguments.length) return height;
    height = value;
    return chart;
  };

  chart.lineColor = function(value) {
    if (!arguments.length) return lineColor;
    lineColor = value;
    return chart;
  };

  return chart;
}

// Usage
const myChart = lineChart()
  .width(600)
  .height(300)
  .lineColor("steelblue");

d3.select("#chart")
  .datum(data)
  .call(myChart);
```

---

## Common Pitfalls

### 1. Data Binding Confusion

**Problem:** Elements not updating correctly

```javascript
// WRONG: No key function
svg.selectAll("circle")
  .data(newData)
  .attr("cx", d => xScale(d.value));

// CORRECT: Handle all cases
const circles = svg.selectAll("circle")
  .data(newData, d => d.id); // Key function!

circles.enter()
  .append("circle")
  .attr("r", 5)
  .merge(circles)
  .attr("cx", d => xScale(d.value));

circles.exit().remove();
```

### 2. Scale Domain/Range Issues

**Problem:** Y-axis starts at wrong value

```javascript
// WRONG: Doesn't start at 0
yScale.domain(d3.extent(data, d => d.value))

// CORRECT: Start at 0 for bar charts
yScale.domain([0, d3.max(data, d => d.value)])

// WRONG: Y-axis inverted
yScale.range([0, height])

// CORRECT: SVG coordinates are top-down
yScale.range([height, 0])
```

### 3. SVG vs Canvas Performance

```javascript
// RULE OF THUMB:
// < 1,000 elements: Use SVG (easier, more features)
// 1,000 - 10,000: Consider Canvas
// > 10,000: Use Canvas or WebGL
```

### 4. Animation Performance

```javascript
// WRONG: Too fast
circles.transition()
  .duration(50)
  .attr("cx", d => xScale(d.value));

// CORRECT: Appropriate duration and easing
circles.transition()
  .duration(300)
  .ease(d3.easeCubicOut)
  .attr("cx", d => xScale(d.value));
```

---

## Integration Patterns

### With React

```javascript
import { useEffect, useRef } from 'react';
import * as d3 from 'd3';

function LineChart({ data, width, height }) {
  const svgRef = useRef();

  useEffect(() => {
    if (!data || !data.length) return;

    const svg = d3.select(svgRef.current);
    svg.selectAll("*").remove(); // Clear previous

    const margin = {top: 20, right: 30, bottom: 40, left: 50};
    const innerWidth = width - margin.left - margin.right;
    const innerHeight = height - margin.top - margin.bottom;

    const g = svg.append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    const xScale = d3.scaleTime()
      .domain(d3.extent(data, d => d.date))
      .range([0, innerWidth]);

    const yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.value)])
      .range([innerHeight, 0]);

    const line = d3.line()
      .x(d => xScale(d.date))
      .y(d => yScale(d.value));

    g.append("path")
      .datum(data)
      .attr("d", line)
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 2);

    g.append("g")
      .attr("transform", `translate(0,${innerHeight})`)
      .call(d3.axisBottom(xScale));

    g.append("g")
      .call(d3.axisLeft(yScale));

  }, [data, width, height]);

  return <svg ref={svgRef} width={width} height={height} />;
}

export default LineChart;
```

### Export to PNG/SVG

```javascript
// Export as SVG
function exportSVG() {
  const svg = document.querySelector("svg");
  const serializer = new XMLSerializer();
  const source = serializer.serializeToString(svg);

  const blob = new Blob([source], {type: "image/svg+xml"});
  const url = URL.createObjectURL(blob);

  const a = document.createElement("a");
  a.href = url;
  a.download = "chart.svg";
  a.click();

  URL.revokeObjectURL(url);
}
```

---

## Resources and References

### Official Documentation
- D3.js API Reference: https://d3js.org/
- Observable Examples: https://observablehq.com/@d3

### Learning Resources
- "Interactive Data Visualization for the Web" by Scott Murray
- D3 Graph Gallery: https://d3-graph-gallery.com/
- Amelia Wattenberger's D3 Tutorial: https://wattenberger.com/blog/d3

### Color Tools
- ColorBrewer: https://colorbrewer2.org/
- D3 Color Schemes: https://d3js.org/d3-scale-chromatic

### Inspiration
- Observable Trending: https://observablehq.com/trending
- Reddit r/dataisbeautiful: https://reddit.com/r/dataisbeautiful

---

This skill provides comprehensive coverage of D3.js for creating professional, interactive data visualizations. Use the examples as starting points and customize them for your specific needs.
