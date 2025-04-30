d3.select(this).select("circle")
            .attr("stroke", matches ? "#333" : "#fff")
            .attr("stroke-width", matches ? 2.5 : 1.5);
            
          d3.select(this).select("text")
            .attr("opacity", matches ? 1 : 0.3);
        });
      });
      
    // Add filter by group
    const filterGroup = controls.append("div")
      .style("display", "flex")
      .style("align-items", "center");
      
    filterGroup.append("label")
      .attr("for", "group-filter")
      .style("margin-right", "5px")
      .text("Group:");
      
    // Get unique groups
    const groups = Array.from(new Set(nodes.map(d => d.group))).sort();
    
    filterGroup.append("select")
      .attr("id", "group-filter")
      .style("padding", "5px")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .on("change", function() {
        const selectedGroup = this.value;
        
        if (selectedGroup === "all") {
          // Show all nodes
          node.style("display", null);
          
          // Show all links
          link.style("display", null);
        } else {
          const groupNumber = parseInt(selectedGroup);
          
          // Filter nodes
          node.style("display", d => 
            d.group === groupNumber ? null : "none");
            
          // Filter links
          link.style("display", d => 
            d.source.group === groupNumber || d.target.group === groupNumber ? null : "none");
        }
      })
      .selectAll("option")
      .data(["all", ...groups])
      .join("option")
        .attr("value", d => d)
        .text(d => d === "all" ? "All Groups" : `Group ${d}`);
        
    // Add layout controls
    const layoutControls = controls.append("div")
      .style("display", "flex")
      .style("gap", "5px");
      
    // Restart force layout
    layoutControls.append("button")
      .attr("class", "restart-button")
      .style("padding", "5px 10px")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .style("background", "#f8f8f8")
      .style("cursor", "pointer")
      .text("Restart Layout")
      .on("click", function() {
        // Reset positions
        nodes.forEach(d => {
          d.fx = null;
          d.fy = null;
        });
        
        // Restart simulation with high alpha
        simulation.alpha(1).restart();
      });
      
    // Center view
    layoutControls.append("button")
      .attr("class", "center-view")
      .style("padding", "5px 10px")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .style("background", "#f8f8f8")
      .style("cursor", "pointer")
      .text("Center View")
      .on("click", function() {
        svg.transition().duration(750).call(
          zoom.transform,
          d3.zoomIdentity.translate(width / 2, height / 2).scale(0.8)
        );
      });
      
    // Export button
    layoutControls.append("button")
      .attr("class", "export-button")
      .style("padding", "5px 10px")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .style("background", "#f8f8f8")
      .style("cursor", "pointer")
      .text("Export SVG")
      .on("click", function() {
        exportSVG();
      });
  }
  
  // Export SVG function
  function exportSVG() {
    // Get SVG element
    const svgEl = svg.node();
    
    // Create a clone for export
    const clone = svgEl.cloneNode(true);
    
    // Convert to string
    const svgString = new XMLSerializer().serializeToString(clone);
    
    // Create data URL
    const svgBlob = new Blob([svgString], {type: "image/svg+xml"});
    const url = URL.createObjectURL(svgBlob);
    
    // Create download link
    const link = document.createElement("a");
    link.href = url;
    link.download = "network_graph.svg";
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    
    // Clean up
    URL.revokeObjectURL(url);
  }
  
  // Add controls to the visualization
  addControls();
  
  // Return public API
  return {
    simulation,
    nodes,
    links,
    selectNode,
    deselectNode,
    deselectAllNodes,
    exportSVG
  };
}

// 4. Animated Chart Transitions with D3
function createAnimatedBarChart(data, container) {
  // Set up dimensions
  const margin = {top: 40, right: 20, bottom: 50, left: 60};
  const width = 800 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;
  
  // Create SVG
  const svg = d3.select(container)
    .append("svg")
    .attr("viewBox", `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
    .attr("preserveAspectRatio", "xMidYMid meet")
    .style("max-width", "100%")
    .style("height", "auto");
    
  // Create chart group
  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);
    
  // Create scales
  const x = d3.scaleBand()
    .domain(data.map(d => d.category))
    .range([0, width])
    .padding(0.2);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value) * 1.1])
    .range([height, 0]);
    
  // Create color scale
  const colorScale = d3.scaleOrdinal()
    .domain(data.map(d => d.category))
    .range(d3.schemeCategory10);
    
  // Add x-axis
  const xAxis = g.append("g")
    .attr("class", "x-axis")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  // Add y-axis
  const yAxis = g.append("g")
    .attr("class", "y-axis")
    .call(d3.axisLeft(y));
    
  // Add x-axis label
  g.append("text")
    .attr("class", "x-axis-label")
    .attr("text-anchor", "middle")
    .attr("x", width / 2)
    .attr("y", height + margin.bottom - 10)
    .text("Category");
    
  // Add y-axis label
  g.append("text")
    .attr("class", "y-axis-label")
    .attr("text-anchor", "middle")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", -margin.left + 15)
    .text("Value");
    
  // Add title
  svg.append("text")
    .attr("class", "chart-title")
    .attr("text-anchor", "middle")
    .attr("x", (width + margin.left + margin.right) / 2)
    .attr("y", margin.top / 2)
    .attr("font-size", "18px")
    .attr("font-weight", "bold")
    .text("Category Values");
    
  // Add bars with initial animation
  const bars = g.selectAll(".bar")
    .data(data)
    .join("rect")
      .attr("class", "bar")
      .attr("x", d => x(d.category))
      .attr("width", x.bandwidth())
      .attr("y", height) // Start from bottom
      .attr("height", 0) // Initial height is 0
      .attr("fill", d => colorScale(d.category))
      .attr("rx", 2) // Rounded corners
      .attr("ry", 2);
      
  // Animate bars on load
  bars.transition()
    .duration(1000)
    .delay((d, i) => i * 100)
    .attr("y", d => y(d.value))
    .attr("height", d => height - y(d.value))
    .attr("opacity", 0.8);
    
  // Add value labels
  const labels = g.selectAll(".value-label")
    .data(data)
    .join("text")
      .attr("class", "value-label")
      .attr("text-anchor", "middle")
      .attr("x", d => x(d.category) + x.bandwidth() / 2)
      .attr("y", d => y(d.value) - 5)
      .attr("font-size", "12px")
      .attr("opacity", 0) // Start invisible
      .text(d => d.value);
      
  // Animate labels
  labels.transition()
    .duration(1000)
    .delay((d, i) => i * 100 + 500) // Delay after bars
    .attr("opacity", 1);
    
  // Function to update chart with new data
  function updateData(newData) {
    // Update scales
    x.domain(newData.map(d => d.category));
    y.domain([0, d3.max(newData, d => d.value) * 1.1]);
    
    // Update x-axis with animation
    xAxis.transition()
      .duration(750)
      .call(d3.axisBottom(x));
      
    // Update y-axis with animation
    yAxis.transition()
      .duration(750)
      .call(d3.axisLeft(y));
      
    // Update bars with animation
    const updatedBars = g.selectAll(".bar")
      .data(newData);
      
    // Remove old bars
    updatedBars.exit()
      .transition()
      .duration(750)
      .attr("y", height)
      .attr("height", 0)
      .attr("opacity", 0)
      .remove();
      
    // Add new bars
    const enterBars = updatedBars.enter()
      .append("rect")
      .attr("class", "bar")
      .attr("x", d => x(d.category))
      .attr("width", x.bandwidth())
      .attr("y", height)
      .attr("height", 0)
      .attr("fill", d => colorScale(d.category))
      .attr("rx", 2)
      .attr("ry", 2)
      .attr("opacity", 0);
      
    // Merge and animate all bars
    enterBars.merge(updatedBars)
      .transition()
      .duration(750)
      .attr("x", d => x(d.category))
      .attr("width", x.bandwidth())
      .attr("y", d => y(d.value))
      .attr("height", d => height - y(d.value))
      .attr("opacity", 0.8);
      
    // Update labels
    const updatedLabels = g.selectAll(".value-label")
      .data(newData);
      
    // Remove old labels
    updatedLabels.exit()
      .transition()
      .duration(750)
      .attr("opacity", 0)
      .remove();
      
    // Add new labels
    const enterLabels = updatedLabels.enter()
      .append("text")
      .attr("class", "value-label")
      .attr("text-anchor", "middle")
      .attr("x", d => x(d.category) + x.bandwidth() / 2)
      .attr("y", d => y(d.value) - 5)
      .attr("font-size", "12px")
      .attr("opacity", 0)
      .text(d => d.value);
      
    // Merge and animate all labels
    enterLabels.merge(updatedLabels)
      .transition()
      .duration(750)
      .attr("x", d => x(d.category) + x.bandwidth() / 2)
      .attr("y", d => y(d.value) - 5)
      .text(d => d.value)
      .attr("opacity", 1);
  }
  
  // Function to sort bars
  function sortBars(order) {
    // Clone data to avoid modifying original
    const sortedData = [...data];
    
    // Sort based on order parameter
    if (order === "ascending") {
      sortedData.sort((a, b) => a.value - b.value);
    } else if (order === "descending") {
      sortedData.sort((a, b) => b.value - a.value);
    } else if (order === "alphabetical") {
      sortedData.sort((a, b) => a.category.localeCompare(b.category));
    }
    
    // Update x scale domain
    x.domain(sortedData.map(d => d.category));
    
    // Update x-axis with animation
    xAxis.transition()
      .duration(750)
      .call(d3.axisBottom(x));
      
    // Animate bars to new positions
    g.selectAll(".bar")
      .data(sortedData, d => d.category)
      .transition()
      .duration(750)
      .attr("x", d => x(d.category));
      
    // Animate labels to new positions
    g.selectAll(".value-label")
      .data(sortedData, d => d.category)
      .transition()
      .duration(750)
      .attr("x", d => x(d.category) + x.bandwidth() / 2);
  }
  
  // Function to change chart type
  function changeChartType(type) {
    if (type === "bar") {
      // Already a bar chart, no change needed
      return;
    } else if (type === "line") {
      // Hide bars and labels
      g.selectAll(".bar")
        .transition()
        .duration(750)
        .attr("opacity", 0);
        
      g.selectAll(".value-label")
        .transition()
        .duration(750)
        .attr("opacity", 0);
        
      // Create line generator
      const line = d3.line()
        .x(d => x(d.category) + x.bandwidth() / 2)
        .y(d => y(d.value))
        .curve(d3.curveMonotoneX);
        
      // Add line path
      const path = g.append("path")
        .datum(data)
        .attr("class", "line")
        .attr("fill", "none")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 2)
        .attr("d", line);
        
      // Animate line drawing
      const pathLength = path.node().getTotalLength();
      path
        .attr("stroke-dasharray", pathLength)
        .attr("stroke-dashoffset", pathLength)
        .transition()
        .duration(1000)
        .attr("stroke-dashoffset", 0);
        
      // Add points
      const points = g.selectAll(".point")
        .data(data)
        .join("circle")
          .attr("class", "point")
          .attr("cx", d => x(d.category) + x.bandwidth() / 2)
          .attr("cy", d => y(d.value))
          .attr("r", 0) // Start with radius 0
          .attr("fill", "steelblue");
          
      // Animate points
      points.transition()
        .duration(750)
        .delay((d, i) => i * 50 + 750) // After line animation
        .attr("r", 5);
    }
  }
  
  // Add animation controls
  const controls = d3.select(container)
    .append("div")
    .attr("class", "chart-controls")
    .style("margin-top", "20px")
    .style("display", "flex")
    .style("gap", "10px");
    
  // Sort controls
  const sortControls = controls.append("div")
    .attr("class", "sort-controls");
    
  sortControls.append("label")
    .attr("for", "sort-options")
    .style("margin-right", "5px")
    .text("Sort:");
    
  sortControls.append("select")
    .attr("id", "sort-options")
    .style("padding", "5px")
    .on("change", function() {
      sortBars(this.value);
    })
    .selectAll("option")
    .data([
      {value: "default", label: "Default"},
      {value: "ascending", label: "Value (Low to High)"},
      {value: "descending", label: "Value (High to Low)"},
      {value: "alphabetical", label: "Alphabetical"}
    ])
    .join("option")
      .attr("value", d => d.value)
      .text(d => d.label);
      
  // Chart type controls
  const typeControls = controls.append("div")
    .attr("class", "type-controls");
    
  typeControls.append("label")
    .attr("for", "chart-type")
    .style("margin-right", "5px")
    .text("Chart Type:");
    
  typeControls.append("select")
    .attr("id", "chart-type")
    .style("padding", "5px")
    .on("change", function() {
      changeChartType(this.value);
    })
    .selectAll("option")
    .data([
      {value: "bar", label: "Bar Chart"},
      {value: "line", label: "Line Chart"}
    ])
    .join("option")
      .attr("value", d => d.value)
      .text(d => d.label);
      
  // Animation button
  controls.append("button")
    .attr("id", "animate-button")
    .style("padding", "5px 10px")
    .text("Replay Animation")
    .on("click", function() {
      // Reset bars
      g.selectAll(".bar")
        .transition()
        .duration(500)
        .attr("y", height)
        .attr("height", 0)
        .attr("opacity", 0)
        .transition()
        .duration(1000)
        .delay((d, i) => i * 100)
        .attr("y", d => y(d.value))
        .attr("height", d => height - y(d.value))
        .attr("opacity", 0.8);
        
      // Reset labels
      g.selectAll(".value-label")
        .transition()
        .duration(500)
        .attr("opacity", 0)
        .transition()
        .duration(1000)
        .delay((d, i) => i * 100 + 500)
        .attr("opacity", 1);
    });
  
  // Return public API
  return {
    updateData,
    sortBars,
    changeChartType
  };
}

// 5. Custom D3 Geovisualization with Interactive Features
function createGeoVisualization(mapData, dataPoints) {
  // Set up dimensions
  const width = 960;
  const height = 500;
  
  // Create SVG
  const svg = d3.select("#geo-viz")
    .append("svg")
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("preserveAspectRatio", "xMidYMid meet")
    .style("max-width", "100%")
    .style("height", "auto");
    
  // Create tooltip
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ddd")
    .style("border-radius", "4px")
    .style("padding", "12px")
    .style("box-shadow", "0 2px 12px rgba(0,0,0,0.15)")
    .style("pointer-events", "none")
    .style("max-width", "250px");
    
  // Create projection
  const projection = d3.geoNaturalEarth1()
    .scale(width / 5.5)
    .translate([width / 2, height / 2]);
    
  // Create path generator
  const path = d3.geoPath()
    .projection(projection);
    
  // Create color scale for map
  const colorScale = d3.scaleLinear()
    .domain([-1, 0, 1])
    .range(["#d73027", "#ffffbf", "#1a9850"])
    .interpolate(d3.interpolateRgb);
    
  // Create color scale for data points
  const pointColorScale = d3.scaleOrdinal()
    .domain(["low", "medium", "high"])
    .range(["#fee08b", "#fdae61", "#d53e4f"]);
    
  // Add zoom behavior
  const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .translateExtent([[0, 0], [width, height]])
    .on("zoom", zoomed);
    
  svg.call(zoom);
  
  // Create map group
  const g = svg.append("g");
  
  // Draw map
  g.selectAll("path")
    .data(mapData.features)
    .join("path")
      .attr("class", "country")
      .attr("d", path)
      .attr("fill", d => d.properties.value ? colorScale(d.properties.value) : "#ccc")
      .attr("stroke", "#fff")
      .attr("stroke-width", 0.5)
      .on("mouseover", function(event, d) {
        // Highlight country
        d3.select(this)
          .attr("stroke", "#333")
          .attr("stroke-width", 1);
          
        // Show tooltip
        tooltip
          .style("visibility", "visible")
          .html(`
            <div style="margin-bottom: 5px;"><strong>${d.properties.name}</strong></div>
            ${d.properties.value !== undefined ? 
              `<div>Value: ${d.properties.value.toFixed(2)}</div>` : 
              '<div>No data available</div>'}
          `)
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mousemove", function(event) {
        // Move tooltip with mouse
        tooltip
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", function() {
        // Reset country style
        d3.select(this)
          .attr("stroke", "#fff")
          .attr("stroke-width", 0.5);
          
        // Hide tooltip
        tooltip
          .style("visibility", "hidden");
      });
      
  // Add data points
  const points = g.selectAll("circle")
    .data(dataPoints)
    .join("circle")
      .attr("class", "data-point")
      .attr("cx", d => projection([d.longitude, d.latitude])[0])
      .attr("cy", d => projection([d.longitude, d.latitude])[1])
      .attr("r", d => Math.sqrt(d.size) * 3)
      .attr("fill", d => pointColorScale(d.category))
      .attr("fill-opacity", 0.7)
      .attr("stroke", "#fff")
      .attr("stroke-width", 0.5)
      .on("mouseover", function(event, d) {
        // Highlight point
        d3.select(this)
          .attr("stroke", "#333")
          .attr("stroke-width", 1.5)
          .attr("fill-opacity", 1);
          
        // Show detailed tooltip
        tooltip
          .style("visibility", "visible")
          .html(`
            <div style="margin-bottom: 5px;"><strong>${d.name}</strong></div>
            <div>Category: ${d.category}</div>
            <div>Size: ${d.size}</div>
            <div>Location: ${d.latitude.toFixed(2)}, ${d.longitude.toFixed(2)}</div>
            ${d.description ? `<div style="margin-top: 5px;">${d.description}</div>` : ""}
          `)
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mousemove", function(event) {
        // Move tooltip with mouse
        tooltip
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", function() {
        // Reset point style
        d3.select(this)
          .attr("stroke", "#fff")
          .attr("stroke-width", 0.5)
          .attr("fill-opacity", 0.7);
          
        // Hide tooltip
        tooltip
          .style("visibility", "hidden");
      })
      .on("click", function(event, d) {
        // Show detailed popup
        showDetailPanel(d);
        
        // Stop propagation
        event.stopPropagation();
      });
      
  // Hide detail panel on map click
  svg.on("click", hideDetailPanel);
  
  // Zoom function
  function zoomed(event) {
    // Transform map
    g.attr("transform", event.transform);
    
    // Adjust stroke width based on zoom level
    g.selectAll(".country")
      .attr("stroke-width", 0.5 / event.transform.k);
      
    // Adjust point size based on zoom level
    g.selectAll(".data-point")
      .attr("r", d => Math.sqrt(d.size) * 3 / Math.sqrt(event.transform.k))
      .attr("stroke-width", 0.5 / event.transform.k);
  }
  
  // Function to show detail panel
  function showDetailPanel(d) {
    // Create or update detail panel
    let detailPanel = d3.select("#detail-panel");
    
    if (detailPanel.empty()) {
      detailPanel = d3.select("body")
        .append("div")
        .attr("id", "detail-panel")
        .style("position", "fixed")
        .style("top", "20px")
        .style("right", "20px")
        .style("width", "300px")
        .style("background", "white")
        .style("border", "1px solid #ddd")
        .style("border-radius", "4px")
        .style("padding", "15px")
        .style("box-shadow", "0 2px 12px rgba(0,0,0,0.15)")
        .style("z-index", "1000")
        .style("max-height", "calc(100vh - 40px)")
        .style("overflow-y", "auto");
    }
    
    // Update panel content
    detailPanel.html(`
      <div style="position: relative;">
        <h3 style="margin-top: 0;">${d.name}</h3>
        <button id="close-panel" style="position: absolute; top: 0; right: 0; background: none; border: none; cursor: pointer; font-size: 16px;">×</button>
        <div style="margin-bottom: 10px;">
          <span style="display: inline-block; width: 12px; height: 12px; border-radius: 50%; background: ${pointColorScale(d.category)}; margin-right: 5px;"></span>
          Category: ${d.category}
        </div>
        <div style="margin-bottom: 10px;">Size: ${d.size}</div>
        <div style="margin-bottom: 10px;">Location: ${d.latitude.toFixed(2)}, ${d.longitude.toFixed(2)}</div>
        ${d.description ? `<p>${d.description}</p>` : ""}
        <div id="point-chart" style="margin-top: 15px; height: 200px;"></div>
      </div>
    `);
    
    // Add close button handler
    detailPanel.select("#close-panel")
      .on("click", function() {
        hideDetailPanel();
      });
      
    // Add small chart (if point has historical data)
    if (d.history) {
      // Create time scale
      const xScale = d3.scaleTime()
        .domain(d3.extent(d.history, d => d.date))
        .range([0, 250]);
        
      // Create value scale
      const yScale = d3.scaleLinear()
        .domain([0, d3.max(d.history, d => d.value) * 1.1])
        .range([150, 0]);
        
      // Create SVG
      const chartSvg = d3.select("#point-chart")
        .append("svg")
        .attr("width", 280)
        .attr("height", 180);
        
      // Add chart group with margins
      const chartG = chartSvg.append("g")
        .attr("transform", "translate(30, 10)");
        
      // Add axes
      chartG.append("g")
        .attr("transform", `translate(0, ${150})`)
        .call(d3.axisBottom(xScale).ticks(4).tickFormat(d3.timeFormat("%b %y")));
        
      chartG.append("g")
        .call(d3.axisLeft(yScale).ticks(5));
        
      // Create line generator
      const line = d3.line()
        .x(d => xScale(d.date))
        .y(d => yScale(d.value))
        .curve(d3.curveMonotoneX);
        
      // Add line path
      const path = chartG.append("path")
        .datum(d.history)
        .attr("fill", "none")
        .attr("stroke", pointColorScale(d.category))
        .attr("stroke-width", 2)
        .attr("d", line);
        
      // Animate line drawing
      const pathLength = path.node().getTotalLength();
      path
        .attr("stroke-dasharray", pathLength)
        .attr("stroke-dashoffset", pathLength)
        .transition()
        .duration(1000)
        .attr("stroke-dashoffset", 0);
        
      // Add points
      chartG.selectAll(".history-point")
        .data(d.history)
        .join("circle")
          .attr("class", "history-point")
          .attr("cx", d => xScale(d.date))### Real-World Examples and Advanced Patterns

```javascript
// 1. Responsive Dashboard Layout
function createResponsiveDashboard() {
  // Dashboard container
  const dashboard = d3.select("#dashboard")
    .style("display", "grid")
    .style("grid-template-columns", "repeat(auto-fit, minmax(300px, 1fr))")
    .style("grid-gap", "20px")
    .style("padding", "20px");
    
  // Add chart containers
  const chartContainers = dashboard.selectAll(".chart-container")
    .data([
      {id: "chart1", title: "Revenue by Category"},
      {id: "chart2", title: "Monthly Trends"},
      {id: "chart3", title: "Regional Distribution"},
      {id: "chart4", title: "Key Metrics"}
    ])
    .join("div")
      .attr("class", "chart-container")
      .attr("id", d => d.id)
      .style("background", "white")
      .style("border-radius", "5px")
      .style("box-shadow", "0 2px 8px rgba(0,0,0,0.1)")
      .style("overflow", "hidden");
      
  // Add headers to chart containers
  chartContainers.append("div")
    .attr("class", "chart-header")
    .style("padding", "15px")
    .style("border-bottom", "1px solid #eee")
    .style("display", "flex")
    .style("justify-content", "space-between")
    .style("align-items", "center")
    .each(function(d) {
      const header = d3.select(this);
      
      // Title
      header.append("h3")
        .style("margin", "0")
        .style("font-size", "16px")
        .text(d.title);
        
      // Controls
      const controls = header.append("div")
        .attr("class", "chart-controls");
        
      // Example: Add dropdown
      if (d.id === "chart1" || d.id === "chart3") {
        controls.append("select")
          .attr("aria-label", "Select time period")
          .style("margin-right", "10px")
          .style("padding", "4px")
          .selectAll("option")
          .data(["Month", "Quarter", "Year"])
          .join("option")
            .attr("value", d => d.toLowerCase())
            .text(d => d);
      }
      
      // Add refresh button for all charts
      controls.append("button")
        .attr("class", "refresh-button")
        .attr("aria-label", "Refresh chart")
        .style("background", "none")
        .style("border", "none")
        .style("cursor", "pointer")
        .html('<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 12a9 9 0 1 1-9-9"></path><path d="M21 3v9h-9"></path></svg>')
        .on("click", function() {
          // Refresh chart data
          updateChart(d.id);
        });
    });
    
  // Add content area for charts
  chartContainers.append("div")
    .attr("class", "chart-content")
    .style("padding", "15px")
    .style("height", "300px");
    
  // Create charts
  createChart1();
  createChart2();
  createChart3();
  createChart4();
  
  // Add global controls
  dashboard.insert("div", ":first-child")
    .attr("class", "dashboard-controls")
    .style("display", "flex")
    .style("justify-content", "space-between")
    .style("margin-bottom", "20px")
    .html(`
      <div class="dashboard-title">
        <h2 style="margin: 0">Sales Performance Dashboard</h2>
        <p style="margin: 5px 0 0 0; color: #666">Last updated: ${new Date().toLocaleString()}</p>
      </div>
      <div class="dashboard-actions">
        <select id="date-range" style="margin-right: 10px; padding: 8px">
          <option value="30">Last 30 Days</option>
          <option value="90">Last 90 Days</option>
          <option value="365">Last Year</option>
          <option value="custom">Custom Range</option>
        </select>
        <button id="export-dashboard" style="padding: 8px 16px; background: #4285F4; color: white; border: none; border-radius: 4px; cursor: pointer">
          Export Dashboard
        </button>
      </div>
    `);
    
  // Add event handlers
  d3.select("#date-range").on("change", function() {
    const value = this.value;
    if (value === "custom") {
      showDateRangePicker();
    } else {
      updateDashboardDateRange(parseInt(value));
    }
  });
  
  d3.select("#export-dashboard").on("click", exportDashboard);
  
  // Handle window resize
  window.addEventListener("resize", debounce(function() {
    // Resize all charts
    resizeCharts();
  }, 250));
  
  // Function to resize charts
  function resizeCharts() {
    // Resize each chart based on container dimensions
    chartContainers.each(function(d) {
      const container = d3.select(this);
      const content = container.select(".chart-content");
      const width = content.node().getBoundingClientRect().width;
      const height = content.node().getBoundingClientRect().height;
      
      // Call resize method for each chart
      if (d.id === "chart1") resizeChart1(width, height);
      if (d.id === "chart2") resizeChart2(width, height);
      if (d.id === "chart3") resizeChart3(width, height);
      if (d.id === "chart4") resizeChart4(width, height);
    });
  }
}

// 2. Advanced Map Visualization with Tooltips and Zoom
function createInteractiveMap() {
  // Set up dimensions
  const width = 960;
  const height = 600;
  
  // Create SVG
  const svg = d3.select("#map")
    .append("svg")
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("preserveAspectRatio", "xMidYMid meet")
    .style("max-width", "100%")
    .style("height", "auto");
    
  // Create tooltip
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ddd")
    .style("border-radius", "4px")
    .style("padding", "12px")
    .style("box-shadow", "0 2px 12px rgba(0,0,0,0.15)")
    .style("pointer-events", "none")
    .style("max-width", "250px");
    
  // Create projection
  const projection = d3.geoMercator()
    .scale(width / 2 / Math.PI)
    .center([0, 0])
    .translate([width / 2, height / 2]);
    
  // Create path generator
  const path = d3.geoPath().projection(projection);
  
  // Add zoom behavior
  const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .translateExtent([[0, 0], [width, height]])
    .on("zoom", zoomed);
    
  svg.call(zoom);
  
  // Create map group
  const g = svg.append("g");
  
  // Load world data (GeoJSON)
  d3.json("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json")
    .then(world => {
      // Convert TopoJSON to GeoJSON
      const countries = topojson.feature(world, world.objects.countries);
      
      // Draw countries
      g.selectAll("path")
        .data(countries.features)
        .join("path")
          .attr("class", "country")
          .attr("d", path)
          .attr("fill", "#ccc")
          .attr("stroke", "#fff")
          .attr("stroke-width", 0.5)
          .on("mouseover", function(event, d) {
            // Highlight country
            d3.select(this)
              .attr("fill", "#9cb6d7");
              
            // Show tooltip
            tooltip
              .style("visibility", "visible")
              .html(`<strong>${d.properties.name}</strong>`)
              .style("left", (event.pageX + 10) + "px")
              .style("top", (event.pageY - 28) + "px");
          })
          .on("mousemove", function(event) {
            // Move tooltip with mouse
            tooltip
              .style("left", (event.pageX + 10) + "px")
              .style("top", (event.pageY - 28) + "px");
          })
          .on("mouseout", function() {
            // Reset country fill
            d3.select(this)
              .attr("fill", "#ccc");
              
            // Hide tooltip
            tooltip
              .style("visibility", "hidden");
          });
          
      // Add country labels
      g.selectAll("text")
        .data(countries.features)
        .join("text")
          .attr("class", "country-label")
          .attr("transform", d => `translate(${path.centroid(d)})`)
          .attr("text-anchor", "middle")
          .attr("font-size", "8px")
          .attr("pointer-events", "none")
          .attr("opacity", 0) // Start hidden
          .text(d => d.properties.name);
    });
    
  // Zoom function
  function zoomed(event) {
    // Transform map
    g.attr("transform", event.transform);
    
    // Adjust stroke width based on zoom level
    g.selectAll(".country")
      .attr("stroke-width", 0.5 / event.transform.k);
      
    // Show labels at higher zoom levels
    g.selectAll(".country-label")
      .attr("opacity", event.transform.k > 3 ? 1 : 0)
      .attr("font-size", `${8 / event.transform.k}px`);
  }
  
  // Add data points
  function addDataPoints(data) {
    // Scale for point size
    const sizeScale = d3.scaleSqrt()
      .domain([0, d3.max(data, d => d.value)])
      .range([3, 20]);
      
    // Color scale for categories
    const colorScale = d3.scaleOrdinal()
      .domain(["A", "B", "C"])
      .range(["#ff9e4a", "#d53e4f", "#3288bd"]);
      
    // Add points
    g.selectAll("circle")
      .data(data)
      .join("circle")
        .attr("cx", d => projection([d.lon, d.lat])[0])
        .attr("cy", d => projection([d.lon, d.lat])[1])
        .attr("r", d => sizeScale(d.value))
        .attr("fill", d => colorScale(d.category))
        .attr("fill-opacity", 0.7)
        .attr("stroke", "#fff")
        .attr("stroke-width", 0.5)
        .on("mouseover", function(event, d) {
          // Highlight point
          d3.select(this)
            .attr("stroke", "#333")
            .attr("stroke-width", 1.5);
            
          // Show detailed tooltip
          tooltip
            .style("visibility", "visible")
            .html(`
              <div style="margin-bottom: 5px;"><strong>${d.name}</strong></div>
              <div>Category: ${d.category}</div>
              <div>Value: ${d.value.toLocaleString()}</div>
              <div>Location: ${d.lat.toFixed(2)}, ${d.lon.toFixed(2)}</div>
            `)
            .style("left", (event.pageX + 10) + "px")
            .style("top", (event.pageY - 28) + "px");
        })
        .on("mousemove", function(event) {
          // Move tooltip with mouse
          tooltip
            .style("left", (event.pageX + 10) + "px")
            .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", function() {
          // Reset point style
          d3.select(this)
            .attr("stroke", "#fff")
            .attr("stroke-width", 0.5);
            
          // Hide tooltip
          tooltip
            .style("visibility", "hidden");
        });
  }
  
  // Add map controls
  function addMapControls() {
    // Create control container
    const controls = d3.select("#map")
      .append("div")
      .attr("class", "map-controls")
      .style("position", "absolute")
      .style("top", "10px")
      .style("right", "10px")
      .style("display", "flex")
      .style("flex-direction", "column")
      .style("gap", "5px");
      
    // Add zoom controls
    controls.append("button")
      .attr("class", "zoom-in")
      .attr("aria-label", "Zoom in")
      .style("width", "30px")
      .style("height", "30px")
      .style("background", "white")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .style("cursor", "pointer")
      .style("font-size", "16px")
      .style("display", "flex")
      .style("align-items", "center")
      .style("justify-content", "center")
      .text("+")
      .on("click", function() {
        svg.transition().duration(300).call(
          zoom.scaleBy, 1.5
        );
      });
      
    controls.append("button")
      .attr("class", "zoom-out")
      .attr("aria-label", "Zoom out")
      .style("width", "30px")
      .style("height", "30px")
      .style("background", "white")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .style("cursor", "pointer")
      .style("font-size", "16px")
      .style("display", "flex")
      .style("align-items", "center")
      .style("justify-content", "center")
      .text("−")
      .on("click", function() {
        svg.transition().duration(300).call(
          zoom.scaleBy, 0.75
        );
      });
      
    controls.append("button")
      .attr("class", "zoom-reset")
      .attr("aria-label", "Reset zoom")
      .style("width", "30px")
      .style("height", "30px")
      .style("background", "white")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .style("cursor", "pointer")
      .style("font-size", "14px")
      .style("display", "flex")
      .style("align-items", "center")
      .style("justify-content", "center")
      .text("↺")
      .on("click", function() {
        svg.transition().duration(750).call(
          zoom.transform,
          d3.zoomIdentity
        );
      });
  }
  
  // Add legend
  function addLegend() {
    // Create legend container
    const legend = d3.select("#map")
      .append("div")
      .attr("class", "map-legend")
      .style("position", "absolute")
      .style("bottom", "20px")
      .style("left", "20px")
      .style("background", "rgba(255, 255, 255, 0.9)")
      .style("border-radius", "4px")
      .style("padding", "10px")
      .style("border", "1px solid #ccc");
      
    // Add legend title
    legend.append("div")
      .style("font-weight", "bold")
      .style("margin-bottom", "5px")
      .text("Data Categories");
      
    // Add legend items
    const items = legend.selectAll(".legend-item")
      .data(["A", "B", "C"])
      .join("div")
        .attr("class", "legend-item")
        .style("display", "flex")
        .style("align-items", "center")
        .style("margin-bottom", "3px");
        
    // Add color swatches
    items.append("div")
      .style("width", "12px")
      .style("height", "12px")
      .style("margin-right", "5px")
      .style("border-radius", "50%")
      .style("background", d => d === "A" ? "#ff9e4a" : d === "B" ? "#d53e4f" : "#3288bd");
      
    // Add labels
    items.append("div")
      .text(d => `Category ${d}`);
  }
  
  return {
    addDataPoints,
    addMapControls,
    addLegend
  };
}

// 3. Interactive Network Graph with Force Simulation
function createNetworkGraph(data) {
  // Set up dimensions
  const width = 800;
  const height = 600;
  
  // Create SVG
  const svg = d3.select("#network")
    .append("svg")
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("preserveAspectRatio", "xMidYMid meet")
    .style("max-width", "100%")
    .style("height", "auto")
    .style("border", "1px solid #ddd");
    
  // Extract nodes and links from data
  const nodes = data.nodes;
  const links = data.links;
  
  // Create color scale
  const color = d3.scaleOrdinal(d3.schemeCategory10);
  
  // Create simulation
  const simulation = d3.forceSimulation(nodes)
    .force("link", d3.forceLink(links).id(d => d.id).distance(100))
    .force("charge", d3.forceManyBody().strength(-300))
    .force("center", d3.forceCenter(width / 2, height / 2))
    .force("collision", d3.forceCollide().radius(d => d.radius || 20));
    
  // Create container group
  const g = svg.append("g");
  
  // Add zoom behavior
  const zoom = d3.zoom()
    .scaleExtent([0.1, 4])
    .on("zoom", event => {
      g.attr("transform", event.transform);
    });
    
  svg.call(zoom);
  
  // Create links
  const link = g.append("g")
    .attr("class", "links")
    .selectAll("line")
    .data(links)
    .join("line")
      .attr("stroke", "#999")
      .attr("stroke-opacity", 0.6)
      .attr("stroke-width", d => Math.sqrt(d.value || 1));
      
  // Create nodes
  const node = g.append("g")
    .attr("class", "nodes")
    .selectAll(".node")
    .data(nodes)
    .join("g")
      .attr("class", "node")
      .call(drag(simulation));
      
  // Add circles to nodes
  node.append("circle")
    .attr("r", d => d.radius || 10)
    .attr("fill", d => color(d.group))
    .attr("stroke", "#fff")
    .attr("stroke-width", 1.5);
    
  // Add labels to nodes
  node.append("text")
    .attr("dx", d => d.radius + 5 || 15)
    .attr("dy", ".35em")
    .text(d => d.name)
    .style("font-size", "10px")
    .style("pointer-events", "none");
    
  // Add title for accessibility
  node.append("title")
    .text(d => d.name);
    
  // Setup tooltip
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ddd")
    .style("border-radius", "4px")
    .style("padding", "12px")
    .style("box-shadow", "0 2px 12px rgba(0,0,0,0.15)")
    .style("pointer-events", "none")
    .style("max-width", "250px");
    
  // Add node interactivity
  node
    .on("mouseover", function(event, d) {
      // Highlight node
      d3.select(this).select("circle")
        .attr("stroke", "#333")
        .attr("stroke-width", 2);
        
      // Highlight connected links and nodes
      link
        .attr("stroke", l => 
          l.source.id === d.id || l.target.id === d.id ? "#333" : "#999")
        .attr("stroke-opacity", l => 
          l.source.id === d.id || l.target.id === d.id ? 1 : 0.2)
        .attr("stroke-width", l => 
          l.source.id === d.id || l.target.id === d.id ? Math.sqrt(l.value || 1) * 2 : Math.sqrt(l.value || 1));
          
      node.select("circle")
        .attr("opacity", n => 
          isConnected(d, n) ? 1 : 0.2);
          
      node.select("text")
        .attr("opacity", n => 
          isConnected(d, n) ? 1 : 0.2);
          
      // Show tooltip
      tooltip
        .style("visibility", "visible")
        .html(`
          <div style="margin-bottom: 5px;"><strong>${d.name}</strong></div>
          <div>Group: ${d.group}</div>
          <div>Connections: ${getConnectionCount(d)}</div>
          ${d.description ? `<div style="margin-top: 5px;">${d.description}</div>` : ""}
        `)
        .style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 28) + "px");
    })
    .on("mousemove", function(event) {
      // Move tooltip with mouse
      tooltip
        .style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 28) + "px");
    })
    .on("mouseout", function() {
      // Reset node style
      d3.select(this).select("circle")
        .attr("stroke", "#fff")
        .attr("stroke-width", 1.5);
        
      // Reset links and nodes
      link
        .attr("stroke", "#999")
        .attr("stroke-opacity", 0.6)
        .attr("stroke-width", d => Math.sqrt(d.value || 1));
        
      node.select("circle")
        .attr("opacity", 1);
        
      node.select("text")
        .attr("opacity", 1);
        
      // Hide tooltip
      tooltip
        .style("visibility", "hidden");
    })
    .on("click", function(event, d) {
      // Select node
      if (d.selected) {
        deselectNode(d);
      } else {
        selectNode(d);
      }
      
      // Stop propagation
      event.stopPropagation();
    });
    
  // Deselect all on background click
  svg.on("click", function() {
    deselectAllNodes();
  });
  
  // Check if two nodes are connected
  function isConnected(a, b) {
    if (a.id === b.id) return true; // Self
    
    return links.some(link => 
      (link.source.id === a.id && link.target.id === b.id) || 
      (link.source.id === b.id && link.target.id === a.id)
    );
  }
  
  // Get connection count for a node
  function getConnectionCount(d) {
    return links.filter(link => 
      link.source.id === d.id || link.target.id === d.id
    ).length;
  }
  
  // Select a node
  function selectNode(d) {
    // Deselect others
    nodes.forEach(n => { n.selected = false; });
    
    // Select this node
    d.selected = true;
    
    // Update node styling
    node.each(function(n) {
      if (n.id === d.id) {
        d3.select(this).select("circle")
          .attr("stroke", "#333")
          .attr("stroke-width", 3)
          .attr("stroke-dasharray", "none");
      } else if (isConnected(d, n)) {
        d3.select(this).select("circle")
          .attr("stroke", "#333")
          .attr("stroke-width", 1.5)
          .attr("stroke-dasharray", "none");
      } else {
        d3.select(this).select("circle")
          .attr("stroke", "#fff")
          .attr("stroke-width", 1.5)
          .attr("stroke-dasharray", "none");
      }
    });
    
    // Update link styling
    link
      .attr("stroke", l => 
        l.source.id === d.id || l.target.id === d.id ? "#333" : "#ccc")
      .attr("stroke-opacity", l => 
        l.source.id === d.id || l.target.id === d.id ? 1 : 0.3)
      .attr("stroke-width", l => 
        l.source.id === d.id || l.target.id === d.id ? Math.sqrt(l.value || 1) * 1.5 : Math.sqrt(l.value || 1));
        
    // Trigger node selection event
    if (typeof onNodeSelect === "function") {
      onNodeSelect(d);
    }
  }
  
  // Deselect a node
  function deselectNode(d) {
    d.selected = false;
    
    // Reset node styling
    node.select("circle")
      .attr("stroke", "#fff")
      .attr("stroke-width", 1.5)
      .attr("stroke-dasharray", "none");
      
    // Reset link styling
    link
      .attr("stroke", "#999")
      .attr("stroke-opacity", 0.6)
      .attr("stroke-width", d => Math.sqrt(d.value || 1));
      
    // Trigger node deselection event
    if (typeof onNodeDeselect === "function") {
      onNodeDeselect(d);
    }
  }
  
  // Deselect all nodes
  function deselectAllNodes() {
    nodes.forEach(n => { n.selected = false; });
    
    // Reset node styling
    node.select("circle")
      .attr("stroke", "#fff")
      .attr("stroke-width", 1.5)
      .attr("stroke-dasharray", "none");
      
    // Reset link styling
    link
      .attr("stroke", "#999")
      .attr("stroke-opacity", 0.6)
      .attr("stroke-width", d => Math.sqrt(d.value || 1));
      
    // Trigger clear selection event
    if (typeof onSelectionClear === "function") {
      onSelectionClear();
    }
  }
  
  // Define drag behavior
  function drag(simulation) {
    function dragstarted(event, d) {
      if (!event.active) simulation.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    }
    
    function dragged(event, d) {
      d.fx = event.x;
      d.fy = event.y;
    }
    
    function dragended(event, d) {
      if (!event.active) simulation.alphaTarget(0);
      
      // Option 1: Keep node position fixed after drag
      // d.fx = d.x;
      // d.fy = d.y;
      
      // Option 2: Allow node to float again
      d.fx = null;
      d.fy = null;
    }
    
    return d3.drag()
      .on("start", dragstarted)
      .on("drag", dragged)
      .on("end", dragended);
  }
  
  // Run simulation
  simulation.on("tick", () => {
    link
      .attr("x1", d => d.source.x)
      .attr("y1", d => d.source.y)
      .attr("x2", d => d.target.x)
      .attr("y2", d => d.target.y);
      
    node
      .attr("transform", d => `translate(${d.x},${d.y})`);
  });
  
  // Add controls
  function addControls() {
    // Create control panel
    const controls = d3.select("#network")
      .append("div")
      .attr("class", "network-controls")
      .style("margin-top", "10px")
      .style("display", "flex")
      .style("gap", "10px")
      .style("flex-wrap", "wrap");
      
    // Add search input
    const search = controls.append("div")
      .style("display", "flex")
      .style("align-items", "center");
      
    search.append("label")
      .attr("for", "node-search")
      .style("margin-right", "5px")
      .text("Search:");
      
    search.append("input")
      .attr("id", "node-search")
      .attr("type", "text")
      .attr("placeholder", "Find node...")
      .style("padding", "5px")
      .style("border", "1px solid #ccc")
      .style("border-radius", "4px")
      .on("input", function() {
        const term = this.value.toLowerCase();
        
        if (term.length < 2) {
          // Reset highlights
          node.select("circle")
            .attr("stroke", "#fff")
            .attr("stroke-width", 1.5);
            
          node.select("text")
            .attr("opacity", 1);
            
          return;
        }
        
        // Highlight matching nodes
        node.each(function(d) {
          const matches = d.name.toLowerCase().includes(term);
          
          d3.select(this).select("circle")
            .attr("stroke", matches ? "#333" : "#fff")
            .attr("stroke### Real-World Examples and Advanced Patterns

```javascript
// 1. Responsive Dashboard Layout
function createResponsiveDashboard() {
  // Dashboard container
  const dashboard = d3.select("#dashboard")
    .style("display", "grid")
    .style("grid-template-columns", "repeat(auto-fit, minmax(300px, 1fr))")
    .style("grid-gap", "20px")
    .style("padding", "20px");
    
  // Add chart containers
  const chartContainers = dashboard.selectAll(".chart-container")
    .data([
      {id: "chart1", title: "Revenue by Category"},
      {id: "chart2", title: "Monthly Trends"},
      {id: "chart3", title: "Regional Distribution"},
      {id: "chart4", title: "Key Metrics"}
    ])
    .join("div")
      .attr("class", "chart-container")
      .attr("id", d => d.id)
      .style("background", "white")
      .style("border-radius", "5px")
      .style("box-shadow", "0 2px 8px rgba(0,0,0,0.1)")
      .style("overflow", "hidden");
      
  // Add headers to chart containers
  chartContainers.append("div")
    .attr("class", "chart-header")
    .style("padding", "15px")
    .style("border-bottom", "1px solid #eee")
    .style("display", "flex")
    .style("justify-content", "space-between")
    .style("align-items", "center")
    .each(function(d) {
      const header = d3.select(this);
      
      // Title
      header.append("h3")
        .style("margin", "0")
        .style("font-size", "16px")
        .text(d.title);
        
      // Controls
      const controls = header.append("div")
        .attr("class", "chart-controls");
        
      // Example: Add dropdown
      if (d.id === "chart1" || d.id === "chart3") {
        controls.append("select")
          .attr("aria-label", "Select time period")
          .style("margin-right", "10px")
          .style("padding", "4px")
          .selectAll("option")
          .data(["Month", "Quarter", "Year"])
          .join("option")
            .attr("value", d => d.toLowerCase())
            .text(d => d);
      }
      
      // Add refresh button for all charts
      controls.append("button")
        .attr("class", "refresh-button")
        .attr("aria-label", "Refresh chart")
        .style("background", "none")
        .style("border", "none")
        .style("cursor", "pointer")
        .html('<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 12a9 9 0 1 1-9-9"></path><path d="M21 3v9h-9"></path></svg>')
        .on("click", function() {
          // Refresh chart data
          updateChart(d.id);
        });
    });
    
  // Add content area for charts
  chartContainers.append("div")
    .attr("class", "chart-content")
    .style("padding", "15px")
    .style("height", "300px");
    
  // Create charts
  createChart1();
  createChart2();
  createChart3();
  createChart4();
  
  // Add global controls
  dashboard.insert("div", ":first-child")
    .attr("class", "dashboard-controls")
    .style("display", "flex")
    .style("justify-content", "space-between")
    .style("margin-bottom", "20px")
    .html(`
      <div class="dashboard-title">
        <h2 style="margin: 0">Sales Performance Dashboard</h2>
        <p style="margin: 5px 0 0 0; color: #666">Last updated: ${new Date().toLocaleString()}</p>
      </div>
      <div class="dashboard-actions">
        <select id="date-range" style="margin-right: 10px; padding: 8px">
          <option value="30">Last 30 Days</option>
          <option value="90">Last 90 Days</option>
          <option value="365">Last Year</option>
          <option value="custom">Custom Range</option>
        </select>
        <button id="export-dashboard" style="padding: 8px 16px; background: #4285F4; color: white; border: none; border-radius: 4px; cursor: pointer">
          Export Dashboard
        </button>
      </div>
    `);
    
  // Add event handlers
  d3.select("#date-range").on("change", function() {
    const value = this.value;
    if (value === "custom") {
      showDateRangePicker();
    } else {
      updateDashboardDateRange(parseInt(value));
    }
  });
  
  d3.select("#export-dashboard").on("click", exportDashboard);
  
  // Handle window resize
  window.addEventListener("resize", debounce(function() {
    // Resize all charts
    resizeCharts();
  }, 250));
  
  // Function to resize charts
  function resizeCharts() {
    // Resize each chart based on container dimensions
    chartContainers.each(function(d) {
      const container = d3.select(this);
      const content = container.select(".chart-content");
      const width = content.node().getBoundingClientRect().width;
      const height = content.node().getBoundingClientRect().height;
      
      // Call resize method for each chart
      if (d.id === "chart1") resizeChart1(width, height);
      if (d.id === "chart2") resizeChart2(width, height);
      if (d.id === "chart3") resizeChart3(width, height);
      if (d.id === "chart4") resizeChart4(width, height);
    });
  }
}

// 2. Advanced Map Visualization with Tooltips and Zoom
function createInteractiveMap() {
  // Set up dimensions
  const width = 960;
  const height = 600;
  
  // Create SVG
  const svg = d3.select("#map")
    .append("svg")
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("preserveAspectRatio", "xMidYMid meet")
    .style("max-width", "100%")
    .style("height", "auto");
    
  // Create tooltip
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ddd")
    .style("border-radius", "4px")
    .style("padding", "12px")
    .style("box-shadow", "0 2px 12px rgba(0,0,0,0.15)")
    .style("pointer-events", "none")
    .style("max-width", "250px");
    
  // Create projection
  const projection = d3.geoMercator()
    .scale(width / 2 / Math.PI)
    .center([0, 0])
    .translate([width / 2, height / 2]);
    
  // Create path generator
  const path = d3.geoPath().projection(projection);
  
  // Add zoom behavior
  const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .translateExtent([[0, 0], [width, height]])
    .on("zoom", zoomed);
    ### Performance Optimization Techniques
```javascript
// 1. Debouncing window resize events
function debounce(func, wait) {
  let timeout;
  return function() {
    const context = this;
    const args = arguments;
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      func.apply(context, args);
    }, wait);
  };
}

window.addEventListener("resize", debounce(function() {
  updateChart();
}, 250));

// 2. Throttling continuous events
function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      func(...args);
    }
  };
}

chart.on("mousemove", throttle(function(event) {
  updateTooltip(event);
}, 50));

// 3. Batch DOM operations
function batchDOMOperations() {
  // Instead of:
  // data.forEach(d => {
  //   const div = document.createElement("div");
  //   div.textContent = d.name;
  //   container.appendChild(div);
  // });
  
  // Better approach:
  const fragment = document.createDocumentFragment();
  data.forEach(d => {
    const div = document.createElement("div");
    div.textContent = d.name;
    fragment.appendChild(div);
  });
  container.appendChild(fragment);
}

// 4. Use requestAnimationFrame for animations
let lastTimestamp = 0;
const animationDuration = 1000; // ms
let startTimestamp = null;

function animate(timestamp) {
  if (!startTimestamp) startTimestamp = timestamp;
  const elapsed = timestamp - startTimestamp;
  const progress = Math.min(elapsed / animationDuration, 1);
  
  // Update visualization based on progress (0 to 1)
  updateVisualization(progress);
  
  // Continue animation if not complete
  if (progress < 1) {
    requestAnimationFrame(animate);
  }
}

// Start animation
requestAnimationFrame(animate);

// 5. Use canvas for very large datasets
function createCanvasScatterplot(data, container, options = {}) {
  // Setup canvas
  const canvas = document.createElement("canvas");
  canvas.width = options.width || 600;
  canvas.height = options.height || 400;
  container.appendChild(canvas);
  
  // Get context
  const ctx = canvas.getContext("2d");
  
  // Create scales (same as with SVG)
  const x = d3.scaleLinear()
    .domain(d3.extent(data, d => d.x))
    .range([0, canvas.width]);
    
  const y = d3.scaleLinear()
    .domain(d3.extent(data, d => d.y))
    .range([canvas.height, 0]);
    
  const color = d3.scaleOrdinal(d3.schemeCategory10);
  
  // Draw points
  function draw() {
    // Clear canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Draw each point
    data.forEach(d => {
      ctx.beginPath();
      ctx.arc(x(d.x), y(d.y), 3, 0, 2 * Math.PI);
      ctx.fillStyle = color(d.category);
      ctx.fill();
    });
    
    // Draw axes (optional)
    drawAxes();
  }
  
  // Handle interactions (more complex with canvas)
  canvas.addEventListener("mousemove", function(event) {
    const rect = canvas.getBoundingClientRect();
    const mouseX = event.clientX - rect.left;
    const mouseY = event.clientY - rect.top;
    
    // Find nearest point
    let minDistance = Infinity;
    let nearestPoint = null;
    
    data.forEach(d => {
      const dx = x(d.x) - mouseX;
      const dy = y(d.y) - mouseY;
      const distance = Math.sqrt(dx * dx + dy * dy);
      
      if (distance < minDistance && distance < 10) {
        minDistance = distance;
        nearestPoint = d;
      }
    });
    
    // Highlight nearest point
    if (nearestPoint) {
      draw(); // Redraw all points
      
      // Draw highlight around nearest point
      ctx.beginPath();
      ctx.arc(x(nearestPoint.x), y(nearestPoint.y), 8, 0, 2 * Math.PI);
      ctx.strokeStyle = "#333";
      ctx.lineWidth = 2;
      ctx.stroke();
      
      // Show tooltip
      showTooltip(nearestPoint, event.clientX, event.clientY);
    } else {
      draw();
      hideTooltip();
    }
  });
  
  // Initial draw
  draw();
  
  return {
    update: function(newData) {
      data = newData;
      draw();
    }
  };
}

// 6. Use Web Workers for heavy computations
// main.js
function setupWebWorker() {
  // Create worker
  const worker = new Worker("worker.js");
  
  // Send data to worker
  worker.postMessage({
    type: "process",
    data: largeDataset
  });
  
  // Receive processed results
  worker.onmessage = function(event) {
    if (event.data.type === "result") {
      // Update visualization with processed data
      updateVisualization(event.data.result);
    } else if (event.data.type === "progress") {
      // Update progress indicator
      updateProgress(event.data.progress);
    }
  };
}

// worker.js
self.onmessage = function(event) {
  if (event.data.type === "process") {
    const data = event.data.data;
    
    // Heavy computation
    const result = processData(data);
    
    // Send back result
    self.postMessage({
      type: "result",
      result: result
    });
  }
};

function processData(data) {
  // Process data in chunks to report progress
  const chunkSize = 1000;
  const chunks = Math.ceil(data.length / chunkSize);
  let processed = [];
  
  for (let i = 0; i < chunks; i++) {
    const chunk = data.slice(i * chunkSize, (i + 1) * chunkSize);
    
    // Process chunk (e.g., clustering, aggregation)
    const processedChunk = chunk.map(transformDataPoint);
    processed = processed.concat(processedChunk);
    
    // Report progress
    self.postMessage({
      type: "progress",
      progress: (i + 1) / chunks
    });
  }
  
  return processed;
}

// 7. Virtual scrolling for large lists
function createVirtualScroller(data, container, rowHeight, renderRow) {
  // Container setup
  container.style.position = "relative";
  container.style.overflow = "auto";
  container.style.height = "500px"; // Fixed height
  
  // Create content container
  const content = document.createElement("div");
  content.style.position = "relative";
  content.style.height = `${data.length * rowHeight}px`; // Total height
  container.appendChild(content);
  
  // Track visible range
  let startIndex = 0;
  let endIndex = 0;
  const visibleRows = {};
  
  // Update visible rows
  function updateVisibleRows() {
    const scrollTop = container.scrollTop;
    const viewportHeight = container.clientHeight;
    
    // Calculate visible range with buffer
    const newStartIndex = Math.floor(scrollTop / rowHeight) - 5;
    const newEndIndex = Math.ceil((scrollTop + viewportHeight) / rowHeight) + 5;
    
    // Adjust indices to be within data range
    const validStartIndex = Math.max(0, newStartIndex);
    const validEndIndex = Math.min(data.length - 1, newEndIndex);
    
    // Remove rows no longer visible
    Object.keys(visibleRows).forEach(index => {
      index = parseInt(index);
      if (index < validStartIndex || index > validEndIndex) {
        if (visibleRows[index].parentNode) {
          content.removeChild(visibleRows[index]);
        }
        delete visibleRows[index];
      }
    });
    
    // Add new visible rows
    for (let i = validStartIndex; i <= validEndIndex; i++) {
      if (!visibleRows[i]) {
        const rowElement = renderRow(data[i], i);
        rowElement.style.position = "absolute";
        rowElement.style.top = `${i * rowHeight}px`;
        rowElement.style.width = "100%";
        rowElement.style.height = `${rowHeight}px`;
        content.appendChild(rowElement);
        visibleRows[i] = rowElement;
      }
    }
    
    startIndex = validStartIndex;
    endIndex = validEndIndex;
  }
  
  // Listen for scroll events
  container.addEventListener("scroll", throttle(updateVisibleRows, 16)); // ~60fps
  
  // Initial update
  updateVisibleRows();
  
  // API
  return {
    refresh: function() {
      // Clear all rows
      Object.keys(visibleRows).forEach(index => {
        if (visibleRows[index].parentNode) {
          content.removeChild(visibleRows[index]);
        }
        delete visibleRows[index];
      });
      
      // Update content height
      content.style.height = `${data.length * rowHeight}px`;
      
      // Refresh visible rows
      updateVisibleRows();
    },
    
    updateData: function(newData) {
      data = newData;
      this.refresh();
    }
  };
}

// 8. Use d3.quadtree for efficient nearest-neighbor queries
function setupQuadtree(data) {
  // Create quadtree
  const quadtree = d3.quadtree()
    .x(d => d.x)
    .y(d => d.y)
    .addAll(data);
    
  // Find closest point
  function findNearest(x, y, radius = Infinity) {
    let closest;
    let closestDistance = radius;
    
    // Search quadtree
    quadtree.visit((node, x0, y0, x1, y1) => {
      if (!node.length) {
        // Leaf node
        const point = node.data;
        const dx = x - point.x;
        const dy = y - point.y;
        const distance = Math.sqrt(dx * dx + dy * dy);
        
        if (distance < closestDistance) {
          closest = point;
          closestDistance = distance;
        }
      }
      
      // Return true to continue traversal
      // Only traverse branches that could contain closer points
      const minDist = node.length ? 
        Math.max(0, 
          Math.min(x - x0, x1 - x) ** 2 + 
          Math.min(y - y0, y1 - y) ** 2) :
        Infinity;
        
      return minDist > closestDistance ** 2;
    });
    
    return closest;
  }
  
  // Find points within radius
  function findPointsWithin(x, y, radius) {
    const points = [];
    const radiusSquared = radius * radius;
    
    quadtree.visit((node, x0, y0, x1, y1) => {
      if (!node.length) {
        // Leaf node
        const point = node.data;
        const dx = x - point.x;
        const dy = y - point.y;
        const distanceSquared = dx * dx + dy * dy;
        
        if (distanceSquared < radiusSquared) {
          points.push(point);
        }
      }
      
      // Check if this quadrant intersects with search circle
      const corners = [
        [x0, y0],
        [x0, y1],
        [x1, y0],
        [x1, y1]
      ];
      
      // If any corner is within radius, we need to explore this quadrant
      const withinRadius = corners.some(corner => {
        const cdx = x - corner[0];
        const cdy = y - corner[1];
        return cdx * cdx + cdy * cdy < radiusSquared;
      });
      
      // Or if the circle intersects with this quadrant's boundaries
      const xOverlap = (x >= x0 - radius && x <= x1 + radius);
      const yOverlap = (y >= y0 - radius && y <= y1 + radius);
      const withinBox = xOverlap && yOverlap;
      
      // Return true to stop traversal of this branch
      return !(withinRadius || withinBox);
    });
    
    return points;
  }
  
  return {
    quadtree,
    findNearest,
    findPointsWithin
  };
}

// 9. Data decimation for large time series
function decimateSeries(data, maxPoints) {
  if (data.length <= maxPoints) {
    return data; // No decimation needed
  }
  
  // Simple method: pick every nth point
  const stride = Math.ceil(data.length / maxPoints);
  const decimated = [];
  
  for (let i = 0; i < data.length; i += stride) {
    decimated.push(data[i]);
  }
  
  // Ensure last point is included (important for trends)
  const lastPoint = data[data.length - 1];
  if (decimated[decimated.length - 1] !== lastPoint) {
    decimated.push(lastPoint);
  }
  
  return decimated;
}

// More advanced: LTTB (Largest-Triangle-Three-Buckets)
function lttbDecimation(data, threshold) {
  const result = [];
  let sampledIndex = 0;
  
  // Always include first point
  result.push(data[0]);
  
  for (let i = 0; i < threshold - 2; i++) {
    // Calculate point score
    const avgRangeStart = Math.floor((i * data.length) / threshold);
    const avgRangeEnd = Math.floor(((i + 1) * data.length) / threshold);
    const avgRangeLength = avgRangeEnd - avgRangeStart;
    const avgRangeSum = data.slice(avgRangeStart, avgRangeEnd)
      .reduce((sum, p) => sum + p.value, 0);
    const avgPoint = avgRangeSum / avgRangeLength;
    
    // Get the range for current bucket
    const rangeStart = Math.floor(((i + 1) * data.length) / threshold);
    const rangeEnd = Math.floor(((i + 2) * data.length) / threshold);
    
    let nextA = 0;
    let maxArea = -1;
    
    for (let j = rangeStart; j < rangeEnd; j++) {
      // Calculate triangle area
      const area = Math.abs(
        (data[sampledIndex].time - data[j].time) * 
        (avgPoint - data[j].value) - 
        (data[sampledIndex].time - avgRangeStart) * 
        (data[j].value - data[j].value)
      ) * 0.5;
      
      if (area > maxArea) {
        maxArea = area;
        nextA = j;
      }
    }
    
    // Select point with largest triangle
    sampledIndex = nextA;
    result.push(data[sampledIndex]);
  }
  
  // Always include last point
  result.push(data[data.length - 1]);
  
  return result;
}

// 10. IndexedDB for client-side data storage
function setupIndexedDB() {
  // Open database
  const request = indexedDB.open("visualizationDB", 1);
  
  // Setup schema
  request.onupgradeneeded = function(event) {
    const db = event.target.result;
    
    // Create object store for data
    const store = db.createObjectStore("datasets", { keyPath: "id" });
    store.createIndex("byName", "name", { unique: false });
    store.createIndex("byDate", "date", { unique: false });
  };
  
  // Handle success
  request.onsuccess = function(event) {
    const db = event.target.result;
    
    // Store data
    function storeData(dataset) {
      const transaction = db.transaction(["datasets"], "readwrite");
      const store = transaction.objectStore("datasets");
      
      // Add or update dataset
      store.put(dataset);
      
      return new Promise((resolve, reject) => {
        transaction.oncomplete = () => resolve(dataset);
        transaction.onerror = event => reject(event.target.error);
      });
    }
    
    // Load data
    function loadData(id) {
      const transaction = db.transaction(["datasets"], "readonly");
      const store = transaction.objectStore("datasets");
      
      // Get dataset by ID
      const request = store.get(id);
      
      return new Promise((resolve, reject) => {
        request.onsuccess = event => resolve(event.target.result);
        request.onerror = event => reject(event.target.error);
      });
    }
    
    // Delete data
    function deleteData(id) {
      const transaction = db.transaction(["datasets"], "readwrite");
      const store = transaction.objectStore("datasets");
      
      // Delete dataset by ID
      const request = store.delete(id);
      
      return new Promise((resolve, reject) => {
        request.onsuccess = () => resolve(true);
        request.onerror = event => reject(event.target.error);
      });
    }
    
    // Load all datasets
    function loadAllData() {
      const transaction = db.transaction(["datasets"], "readonly");
      const store = transaction.objectStore("datasets");
      
      // Get all datasets
      const request = store.getAll();
      
      return new Promise((resolve, reject) => {
        request.onsuccess = event => resolve(event.target.result);
        request.onerror = event => reject(event.target.error);
      });
    }
    
    // Usage example
    async function example() {
      // Store dataset
      await storeData({
        id: "population-2022",
        name: "Population Data 2022",
        date: new Date(),
        data: populationData
      });
      
      // Load dataset
      const dataset = await loadData("population-2022");
      
      // Use data to create visualization
      createVisualization(dataset.data);
    }
  };
}

// 11. Using Typed Arrays for large numeric datasets
function useTypedArrays(rawData) {
  // Convert JSON data to typed arrays for efficiency
  const n = rawData.length;
  
  // Create typed arrays for each numeric property
  const xValues = new Float32Array(n);
  const yValues = new Float32Array(n);
  const values = new Float32Array(n);
  
  // Store string data separately
  const categories = [];
  const categoryMap = new Map();
  const categoryIndices = new Uint16Array(n);
  
  // Convert data
  rawData.forEach((d, i) => {
    xValues[i] = d.x;
    yValues[i] = d.y;
    values[i] = d.value;
    
    // Convert categorical data to indices
    if (!categoryMap.has(d.category)) {
      categoryMap.set(d.category, categories.length);
      categories.push(d.category);
    }
    
    categoryIndices[i] = categoryMap.get(d.category);
  });
  
  // Return typed data
  return {
    length: n,
    x: xValues,
    y: yValues,
    value: values,
    categoryIndex: categoryIndices,
    categories: categories,
    getCategoryName: index => categories[index]
  };
}

// Access typed array data
function accessTypedData(typedData, index) {
  return {
    x: typedData.x[index],
    y: typedData.y[index],
    value: typedData.value[index],
    category: typedData.getCategoryName(typedData.categoryIndex[index])
  };
}

// Using typed arrays for rendering
function renderWithTypedArrays(typedData, ctx) {
  const n = typedData.length;
  
  // Clear canvas
  ctx.clearRect(0, 0, width, height);
  
  // Render points
  for (let i = 0; i < n; i++) {
    const x = xScale(typedData.x[i]);
    const y = yScale(typedData.y[i]);
    const category = typedData.categoryIndex[i];
    
    ctx.beginPath();
    ctx.arc(x, y, 3, 0, 2 * Math.PI);
    ctx.fillStyle = colorScale(category);
    ctx.fill();
  }
}
```### Accessibility and Best Practices
```javascript
// 1. Make visualizations accessible
function createAccessibleChart() {
  // Add proper ARIA roles
  svg.attr("role", "img")
    .attr("aria-labelledby", "chart-title chart-desc");
    
  // Add title and description for screen readers
  svg.append("title")
    .attr("id", "chart-title")
    .text("Annual Revenue by Category (2020-2024)");
    
  svg.append("desc")
    .attr("id", "chart-desc")
    .text("Bar chart showing annual revenue growth across five product categories from 2020 to 2024. Technology shows the highest growth, followed by Services.");
    
  // Add text alternatives for data points
  svg.selectAll(".bar")
    .attr("aria-label", d => `${d.category}: ${d.value} million in revenue for ${d.year}`);
    
  // Create keyboard navigable elements with tabindex
  svg.selectAll(".interactive-element")
    .attr("tabindex", 0)  // Makes elements focusable
    .attr("role", "button")
    .attr("aria-label", d => `Show details for ${d.name}`)
    .on("keydown", function(event, d) {
      // Handle keyboard interactions
      if (event.key === "Enter" || event.key === " ") {
        event.preventDefault();
        // Trigger the same action as a click
        showDetails(d);
      }
    });
    
  // Add keyboard event listeners for chart-wide navigation
  d3.select(svg.node().parentNode)
    .attr("tabindex", 0)
    .on("keydown", function(event) {
      switch(event.key) {
        case "ArrowRight":
          // Move to next data point
          navigateChart("next");
          break;
        case "ArrowLeft":
          // Move to previous data point
          navigateChart("prev");
          break;
        case "Home":
          // Jump to first data point
          navigateChart("first");
          break;
        case "End":
          // Jump to last data point
          navigateChart("last");
          break;
      }
    });
    
  // Ensure sufficient color contrast
  // Use colorblind-friendly palettes
  const accessibleColorScale = d3.scaleOrdinal()
    .domain(categories)
    .range([
      "#0077BB", // Blue
      "#33BBEE", // Cyan
      "#009988", // Teal
      "#EE7733", // Orange
      "#CC3311", // Red
      "#EE3377", // Magenta
      "#BBBBBB"  // Grey
    ]);
    
  // Use patterns in addition to colors
  const patterns = svg.append("defs")
    .selectAll("pattern")
    .data(categories)
    .join("pattern")
      .attr("id", d => `pattern-${d.replace(/\s+/g, '-').toLowerCase()}`)
      .attr("patternUnits", "userSpaceOnUse")
      .attr("width", 10)
      .attr("height", 10)
      .attr("patternTransform", (d, i) => `rotate(${i * 45})`)
      .each(function(d, i) {
        const pattern = d3.select(this);
        
        // Create different patterns based on index
        switch(i % 5) {
          case 0: // Diagonal lines
            pattern.append("rect")
              .attr("width", 10)
              .attr("height", 10)
              .attr("fill", accessibleColorScale(d));
              
            pattern.append("path")
              .attr("d", "M-1,1 l2,-2 M0,10 l10,-10 M9,11 l2,-2")
              .attr("stroke", "#fff")
              .attr("stroke-width", 1.5);
            break;
          case 1: // Dots
            pattern.append("rect")
              .attr("width", 10)
              .attr("height", 10)
              .attr("fill", accessibleColorScale(d));
              
            pattern.append("circle")
              .attr("cx", 3)
              .attr("cy", 3)
              .attr("r", 1.5)
              .attr("fill", "#fff");
              
            pattern.append("circle")
              .attr("cx", 7)
              .attr("cy", 7)
              .attr("r", 1.5)
              .attr("fill", "#fff");
            break;
          // Add more pattern variations
        }## Best Practices and Advanced Techniques

### Performance Optimization
```javascript
// 1. Use SVG clipping paths for large datasets
svg.append("defs").append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
    .attr("width", width)
    .attr("height", height);

chart.attr("clip-path", "url(#chart-area)");

// 2. Use canvas for very large datasets (thousands of elements)
const canvas = d3.select("#chart")
  .append("canvas")
  .attr("width", width)
  .attr("height", height);
  
const context = canvas.node().getContext("2d");

// Draw data points on canvas
data.forEach(d => {
  context.fillStyle = colorScale(d.category);
  context.beginPath();
  context.arc(x(d.x), y(d.y), 3, 0, 2 * Math.PI);
  context.fill();
});

// 3. Use requestAnimationFrame for smooth animations
function animate() {
  // Update data
  data = updateData(data);
  
  // Update elements
  selection.data(data)
    .attr("cx", d => x(d.x))
    .attr("cy", d => y(d.y));
    
  // Request next frame
  requestAnimationFrame(animate);
}

// Start animation
requestAnimationFrame(animate);

// 4. Data filtering and decimation
// Reduce points when zoomed out
function filterData(data, zoom) {
  const factor = Math.ceil(data.length / (width / zoom.k / 5));
  return factor <= 1 ? data : data.filter((d, i) => i % factor === 0);
}

// 5. Efficient update patterns
function efficientUpdate(data) {
  // Only select/modify what needs to change
  // Bad: selectAll inside loop
  // for (let i = 0; i < data.length; i++) {
  //   d3.selectAll("circle").filter((d, j) => j === i)
  //     .attr("r", data[i].radius);
  // }
  
  // Good: Set data once, update all
  const circles = svg.selectAll("circle")
    .data(data);
    
  circles.attr("r", d => d.radius);
}

// 6. Throttle/debounce event handlers
function throttle(callback, delay) {
  let last = 0;
  return function(...args) {
    const now = Date.now();
    if (now - last >= delay) {
      callback.apply(this, args);
      last = now;
    }
  };
}

svg.call(zoom)
  .on("wheel", throttle(function(event) {
    // Handle zoom
  }, 50));
```

### Reusable Components
```javascript
// Create a reusable bar chart component
function barChart() {
  // Default values
  let width = 600;
  let height = 400;
  let margin = {top: 20, right: 20, bottom: 30, left: 40};
  let xValue = d => d.x;
  let yValue = d => d.y;
  let color = "steelblue";
  let duration = 500;
  
  // Create chart function
  function chart(selection) {
    selection.each(function(data) {
      // Create or select SVG element
      const svg = d3.select(this).selectAll("svg")
        .data([data])
        .join("svg")
          .attr("width", width)
          .attr("height", height)
          .attr("viewBox", `0 0 ${width} ${height}`)
          .attr("style", "max-width: 100%; height: auto;");
          
      // Create or select container group
      const g = svg.selectAll(".container")
        .data([data])
        .join("g")
          .attr("class", "container")
          .attr("transform", `translate(${margin.left},${margin.top})`);
          
      // Calculate inner dimensions
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;
      
      // Create scales
      const x = d3.scaleBand()
        .domain(data.map(xValue))
        .range([0, innerWidth])
        .padding(0.1);
        
      const y = d3.scaleLinear()
        .domain([0, d3.max(data, yValue)])
        .nice()
        .range([innerHeight, 0]);
        
      // Create or update axes
      const xAxis = g.selectAll(".x-axis")
        .data([data])
        .join("g")
          .attr("class", "x-axis")
          .attr("transform", `translate(0,${innerHeight})`)
          .transition()
          .duration(duration)
          .call(d3.axisBottom(x));
          
      const yAxis = g.selectAll(".y-axis")
        .data([data])
        .join("g")
          .attr("class", "y-axis")
          .transition()
          .duration(duration)
          .call(d3.axisLeft(y));
          
      // Create or update bars
      const bars = g.selectAll(".bar")
        .data(data)
        .join(
          enter => enter.append("rect")
            .attr("class", "bar")
            .attr("x", d => x(xValue(d)))
            .attr("width", x.bandwidth())
            .attr("y", innerHeight)
            .attr("height", 0)
            .attr("fill", color)
            .call(enter => enter.transition()
              .duration(duration)
              .attr("y", d => y(yValue(d)))
              .attr("height", d => innerHeight - y(yValue(d)))
            ),
          update => update.transition()
            .duration(duration)
            .attr("x", d => x(xValue(d)))
            .attr("width", x.bandwidth())
            .attr("y", d => y(yValue(d)))
            .attr("height", d => innerHeight - y(yValue(d))),
          exit => exit.transition()
            .duration(duration)
            .attr("y", innerHeight)
            .attr("height", 0)
            .remove()
        );
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
  
  chart.margin = function(value) {
    if (!arguments.length) return margin;
    margin = {...margin, ...value};
    return chart;
  };
  
  chart.x = function(value) {
    if (!arguments.length) return xValue;
    xValue = value;
    return chart;
  };
  
  chart.y = function(value) {
    if (!arguments.length) return yValue;
    yValue = value;
    return chart;
  };
  
  chart.color = function(value) {
    if (!arguments.length) return color;
    color = value;
    return chart;
  };
  
  chart.duration = function(value) {
    if (!arguments.length) return duration;
    duration = value;
    return chart;
  };
  
  return chart;
}

// Usage:
const myChart = barChart()
  .width(800)
  .height(500)
  .margin({top: 30, right: 30, bottom: 40, left: 50})
  .x(d => d.name)
  .y(d => d.value)
  .color("#4682b4")
  .duration(750);
  
d3.select("#chart")
  .datum(data)
  .call(myChart);
```

### Advanced Data Visualization Patterns
```javascript
// 1. Observable Pattern for Linked Views
function createLinkedViews() {
  // Create a dispatch to handle events
  const dispatch = d3.dispatch("highlight", "select", "filter");
  
  // Create charts
  const scatterplot = createScatterplot()
    .on("highlight", id => {
      dispatch.call("highlight", null, id);
    })
    .on("select", id => {
      dispatch.call("select", null, id);
    });
    
  const histogram = createHistogram()
    .on("filter", range => {
      dispatch.call("filter", null, range);
    });
    
  // Set up event listeners
  dispatch.on("highlight", id => {
    scatterplot.highlight(id);
    histogram.highlight(id);
  });
  
  dispatch.on("select", id => {
    scatterplot.select(id);
    histogram.select(id);
  });
  
  dispatch.on("filter", range => {
    scatterplot.filter(range);
  });
  
  return {
    scatterplot,
    histogram,
    dispatch
  };
}

// 2. Data Joins for Complex Visualizations
function createComplexViz(data) {
  // Group data
  const nestedData = d3.group(data, d => d.category);
  
  // First-level join - categories
  const categories = svg.selectAll(".category")
    .data(Array.from(nestedData.entries()))
    .join("g")
      .attr("class", d => `category category-${d[0]}`)
      .attr("transform", (d, i) => `translate(0,${i * 100})`);
      
  // Add category labels
  categories.selectAll(".category-label")
    .data(d => [d])
    .join("text")
      .attr("class", "category-label")
      .attr("x", 0)
      .attr("y", -10)
      .text(d => d[0]);
      
  // Second-level join - items within categories
  const items = categories.selectAll(".item")
    .data(d => d[1].map(item => ({
      category: d[0],
      ...item
    })))
    .join("g")
      .attr("class", "item")
      .attr("transform", (d, i) => `translate(${i * 30},0)`);
      
  // Add item circles
  items.selectAll("circle")
    .data(d => [d])
    .join("circle")
      .attr("r", d => sizeScale(d.value))
      .attr("fill", d => colorScale(d.type));
      
  // Add item labels
  items.selectAll(".item-label")
    .data(d => [d])
    .join("text")
      .attr("class", "item-label")
      .attr("text-anchor", "middle")
      .attr("dy", d => sizeScale(d.value) + 12)
      .text(d => d.id);
}

// 3. Advanced Tooltips
function createAdvancedTooltip() {
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ddd")
    .style("border-radius", "3px")
    .style("padding", "10px")
    .style("box-shadow", "0 2px 4px rgba(0,0,0,0.1)")
    .style("pointer-events", "none")
    .style("font-family", "sans-serif")
    .style("font-size", "12px");
    
  // Add tooltip header
  tooltip.append("div")
    .attr("class", "tooltip-header")
    .style("font-weight", "bold")
    .style("margin-bottom", "5px");
    
  // Add tooltip content
  tooltip.append("div")
    .attr("class", "tooltip-content");
    
  // Add small chart inside tooltip
  const chartWidth = 150;
  const chartHeight = 50;
  
  const tooltipSvg = tooltip.append("svg")
    .attr("width", chartWidth)
    .attr("height", chartHeight)
    .style("margin-top", "5px");
    
  const tooltipX = d3.scaleLinear()
    .range([0, chartWidth]);
    
  const tooltipY = d3.scaleLinear()
    .range([chartHeight, 0]);
    
  const tooltipLine = d3.line()
    .x((d, i) => tooltipX(i))
    .y(d => tooltipY(d));
    
  const tooltipPath = tooltipSvg.append("path")
    .attr("fill", "none")
    .attr("stroke", "#69b3a2")
    .attr("stroke-width", 1.5);
    
  // Tooltip show function
  function show(event, d) {
    // Position tooltip
    tooltip
      .style("visibility", "visible")
      .style("left", `${event.pageX + 10}px`)
      .style("top", `${event.pageY - 10}px`);
      
    // Update tooltip content
    tooltip.select(".tooltip-header")
      .text(`${d.name} (${d.category})`);
      
    tooltip.select(".tooltip-content")
      .html(`Value: <strong>${d.value}</strong><br>
             Rank: <strong>${d.rank}</strong>`);
             
    // Update tooltip chart
    if (d.history) {
      tooltipX.domain([0, d.history.length - 1]);
      tooltipY.domain([0, d3.max(d.history)]).nice();
      
      tooltipPath.datum(d.history)
        .attr("d", tooltipLine);
    } else {
      tooltipSvg.style("display", "none");
    }
  }
  
  // Tooltip move function
  function move(event) {
    tooltip
      .style("left", `${event.pageX + 10}px`)
      .style("top", `${event.pageY - 10}px`);
  }
  
  // Tooltip hide function
  function hide() {
    tooltip.style("visibility", "hidden");
  }
  
  return {
    show,
    move,
    hide
  };
}

// 4. Time-series with focus+context (brush and zoom)
function createTimeseriesWithFocus(data) {
  // Set up dimensions
  const margin = {top: 20, right: 20, bottom: 100, left: 50};
  const margin2 = {top: 430, right: 20, bottom: 30, left: 50};
  const width = 960 - margin.left - margin.right;
  const height = 500 - margin.top - margin.bottom;
  const height2 = 500 - margin2.top - margin2.bottom;
  
  // Create scales
  const x = d3.scaleTime()
    .domain(d3.extent(data, d => d.date))
    .range([0, width]);
    
  const x2 = d3.scaleTime()
    .domain(x.domain())
    .range([0, width]);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .nice()
    .range([height, 0]);
    
  const y2 = d3.scaleLinear()
    .domain(y.domain())
    .range([height2, 0]);
    
  // Create line generators
  const line = d3.line()
    .x(d => x(d.date))
    .y(d => y(d.value));
    
  const line2 = d3.line()
    .x(d => x2(d.date))
    .y(d => y2(d.value));
    
  // Create SVG
  const svg = d3.select("#chart").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom);
    
  // Clip path for main chart
  svg.append("defs").append("clipPath")
    .attr("id", "clip")
    .append("rect")
      .attr("width", width)
      .attr("height", height);
      
  // Main chart group
  const focus = svg.append("g")
    .attr("class", "focus")
    .attr("transform", `translate(${margin.left},${margin.top})`);
    
  // Context (small) chart group
  const context = svg.append("g")
    .attr("class", "context")
    .attr("transform", `translate(${margin2.left},${margin2.top})`);
    
  // Add main chart path
  focus.append("path")
    .datum(data)
    .attr("class", "line")
    .attr("clip-path", "url(#clip)")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1.5)
    .attr("d", line);
    
  // Add main chart axes
  focus.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  focus.append("g")
    .attr("class", "axis axis--y")
    .call(d3.axisLeft(y));
    
  // Add context chart path
  context.append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1)
    .attr("d", line2);
    
  // Add context chart axis
  context.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height2})`)
    .call(d3.axisBottom(x2));
    
  // Add brush
  const brush = d3.brushX()
    .extent([[0, 0], [width, height2]])
    .on("brush end", brushed);
    
  context.append("g")
    .attr("class", "brush")
    .call(brush)
    .call(brush.move, x.range());
    
  // Add zoom
  const zoom = d3.zoom()
    .scaleExtent([1, Infinity])
    .translateExtent([[0, 0], [width, height]])
    .extent([[0, 0], [width, height]])
    .on("zoom", zoomed);
    
  svg.append("rect")
    .attr("class", "zoom")
    .attr("width", width)
    .attr("height", height)
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .attr("fill", "none")
    .attr("pointer-events", "all")
    .call(zoom);
    
  // Brush handler
  function brushed(event) {
    if (event.sourceEvent && event.sourceEvent.type === "zoom") return;
    
    const selection = event.selection || x2.range();
    x.domain(selection.map(x2.invert, x2));
    
    focus.select(".line")
      .attr("d", line);
      
    focus.select(".axis--x")
      .call(d3.axisBottom(x));
      
    svg.select(".zoom")
      .call(zoom.transform, d3.zoomIdentity
        .scale(width / (selection[1] - selection[0]))
        .translate(-selection[0], 0));
  }
  
  // Zoom handler
  function zoomed(event) {
    if (event.sourceEvent && event.sourceEvent.type === "brush") return;
    
    const transform = event.transform;
    x.domain(transform.rescaleX(x2).domain());
    
    focus.select(".line")
      .attr("d", line);
      
    focus.select(".axis--x")
      .call(d3.axisBottom(x));
      
    context.select(".brush")
      .call(brush.move, x.range().map(transform.invertX, transform));
  }
}

// 5. Voronoi overlay for enhanced interaction
function createVoronoiInteraction(points) {
  // Create Delaunay triangulation
  const delaunay = d3.Delaunay.from(
    points,
    d => x(d.x),
    d => y(d.y)
  );
  
  // Create Voronoi diagram
  const voronoi = delaunay.voronoi([0, 0, width, height]);
  
  // Add invisible Voronoi cells for interaction
  svg.append("g")
    .attr("class", "voronoi")
    .selectAll("path")
    .data(points)
    .join("path")
      .attr("d", (d, i) => voronoi.renderCell(i))
      .attr("fill", "none")
      .attr("pointer-events", "all")
      .on("mouseover", (event, d) => {
        // Highlight corresponding point
        svg.selectAll(".point")
          .attr("fill-opacity", p => p === d ? 1 : 0.3)
          .attr("r", p => p === d ? 8 : 3);
          
        // Show tooltip
        tooltip.show(event, d);
      })
      .on("mouseout", () => {
        // Reset points
        svg.selectAll(".point")
          .attr("fill-opacity", 0.7)
          .attr("r", 3);
          
        // Hide tooltip
        tooltip.hide();
      });
      
  // Debug visualization (optional)
  // svg.select(".voronoi").selectAll("path")
  //   .attr("stroke", "#ccc")
  //   .attr("fill", "none")
  //   .attr("stroke-opacity", 0.2);
}

// 6. Responsive design pattern
function createResponsiveChart() {
  // Base dimensions
  const aspectRatio = 0.6; // height = width * aspectRatio
  
  // Get container width
  const container = d3.select("#chart");
  const containerWidth = container.node().getBoundingClientRect().width;
  
  // Calculate dimensions
  let width = containerWidth;
  let height = width * aspectRatio;
  
  // Create responsive SVG
  const svg = container.append("svg")
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("style", "width: 100%; height: auto;");
    
  // Function to update chart on resize
  function resize() {
    // Get new container width
    const newWidth = container.node().getBoundingClientRect().width;
    
    // Update dimensions
    width = newWidth;
    height = width * aspectRatio;
    
    // Update SVG viewBox
    svg.attr("viewBox", `0 0 ${width} ${height}`);
    
    // Update scales
    x.range([margin.left, width - margin.right]);
    y.range([height - margin.bottom, margin.top]);
    
    // Update chart elements
    updateChart();
  }
  
  // Add resize listener
  window.addEventListener("resize", debounce(resize, 250));
  
  // Debounce function
  function debounce(func, wait) {
    let timeout;
    return function() {
      const context = this, args = arguments;
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        func.apply(context, args);
      }, wait);
    };
  }
  
  // Function to update chart elements
  function updateChart() {
    // Update axes
    svg.select(".x-axis")
      .attr("transform", `translate(0,${height - margin.bottom})`)
      .call(d3.axisBottom(x));
      
    svg.select(".y-axis")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(y));
      
    // Update other elements...
  }
}

// 7. Geographic projections and transformations
function createGeoMap() {
  // World map data (typically loaded via d3.json)
  const world = {/* TopoJSON data */};
  
  // Create projection
  const projection = d3.geoNaturalEarth1()
    .fitSize([width, height], topojson.feature(world, world.objects.countries));
    
  // Create path generator
  const path = d3.geoPath()
    .projection(projection);
    
  // Draw countries
  svg.selectAll("path.country")
    .data(topojson.feature(world, world.objects.countries).features)
    .join("path")
      .attr("class", "country")
      .attr("d", path)
      .attr("fill", "#ccc")
      .attr("stroke", "#fff")
      .attr("stroke-width", 0.5);
      
  // Add graticule for reference lines
  const graticule = d3.geoGraticule();
  
  svg.append("path")
    .datum(graticule)
    .attr("class", "graticule")
    .attr("d", path)
    .attr("fill", "none")
    .attr("stroke", "#ccc")
    .attr("stroke-width", 0.2)
    .attr("stroke-dasharray", "2,2");
    
  // Add labels at country centroids
  svg.selectAll("text.country-label")
    .data(topojson.feature(world, world.objects.countries).features)
    .join("text")
      .attr("class", "country-label")
      .attr("transform", d => `translate(${path.centroid(d)})`)
      .attr("text-anchor", "middle")
      .attr("font-size", "8px")
      .text(d => d.properties.name);
      
  // Add zoom and pan behavior for maps
  const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .on("zoom", event => {
      svg.selectAll("path.country, path.graticule")
        .attr("transform", event.transform);
        
      svg.selectAll("text.country-label")
        .attr("transform", d => {
          const [x, y] = path.centroid(d);
          return `translate(${x * event.transform.k + event.transform.x}, ${y * event.transform.k + event.transform.y})`;
        })
        .attr("font-size", `${8 / event.transform.k}px`);
    });
    
  svg.call(zoom);
}

// 8. Advanced animations with interpolation
function animateTransition(oldData, newData) {
  // Create interpolator function for each data point
  const interpolators = oldData.map((d, i) => {
    const newD = newData.find(nd => nd.id === d.id) || {x: d.x, y: 0};
    return {
      x: d3.interpolateNumber(d.x, newD.x),
      y: d3.interpolateNumber(d.y, newD.y),
      radius: d3.interpolateNumber(d.radius, newD.radius || 5),
      color: d3.interpolateRgb(d.color, newD.color || "#ccc")
    };
  });
  
  // New data points to add (not in old data)
  const newPoints = newData.filter(d => !oldData.some(od => od.id === d.id));
  
  // Animation function
  function animate(t) {
    // Calculate interpolated values
    const interpolated = oldData.map((d, i) => ({
      id: d.id,
      x: interpolators[i].x(t),
      y: interpolators[i].y(t),
      radius: interpolators[i].radius(t),
      color: interpolators[i].color(t)
    }));
    
    // If at end of transition, include new points
    if (t === 1) {
      interpolated.push(...newPoints);
    }
    
    // Update visualization with interpolated data
    updateViz(interpolated);
    
    // Continue animation if not complete
    if (t < 1) {
      requestAnimationFrame(() => animate(Math.min(1, t + 0.02)));
    }
  }
  
  // Start animation
  requestAnimationFrame(() => animate(0));
  
  // Function to update visualization
  function updateViz(data) {
    svg.selectAll("circle")
      .data(data, d => d.id)
      .join(
        enter => enter.append("circle")
          .attr("cx", d => x(d.x))
          .attr("cy", height)
          .attr("r", 0)
          .attr("fill", d => d.color)
          .call(enter => enter.transition()
            .duration(500)
            .attr("cy", d => y(d.y))
            .attr("r", d => d.radius)
          ),
        update => update
          .attr("cx", d => x(d.x))
          .attr("cy", d => y(d.y))
          .attr("r", d => d.radius)
          .attr("fill", d => d.color),
        exit => exit.transition()
          .duration(500)
          .attr("cy", height)
          .attr("r", 0)
          .remove()
      );
  }
}

// 9. Custom transitions with FLIP animation
function flipAnimation(selection, targetLayout) {
  // First: capture the current positions
  const elements = selection.nodes();
  const firstPositions = elements.map(el => {
    const rect = el.getBoundingClientRect();
    return {
      left: rect.left,
      top: rect.top,
      width: rect.width,
      height: rect.height
    };
  });
  
  // Last: apply the new layout immediately
  targetLayout();
  
  // Invert: calculate the differences and apply transforms
  elements.forEach((el, i) => {
    const first = firstPositions[i];
    const last = el.getBoundingClientRect();
    
    // Calculate transforms
    const dx = first.left - last.left;
    const dy = first.top - last.top;
    const scaleX = first.width / last.width;
    const scaleY = first.height / last.height;
    
    // Apply initial transform to invert the change
    el.style.transform = `translate(${dx}px, ${dy}px) scale(${scaleX}, ${scaleY})`;
    el.style.transformOrigin = "top left";
  });
  
  // Play: animate to identity transform
  requestAnimationFrame(() => {
    elements.forEach(el => {
      el.style.transition = "transform 0.5s ease-out";
      el.style.transform = "translate(0, 0) scale(1, 1)";
    });
  });
}

// 10. Advanced interactivity with filters and cross-filtering
function createCrossFilter(data) {
  // Create crossfilter instance
  const cf = crossfilter(data);
  
  // Create dimensions
  const byCategory = cf.dimension(d => d.category);
  const byValue = cf.dimension(d => d.value);
  const byDate = cf.dimension(d => d.date);
  
  // Create bar chart for categories
  const categoryChart = barChart()
    .dimension(byCategory)
    .group(byCategory.group().reduceCount())
    .x(d3.scaleBand())
    .y(d3.scaleLinear())
    .on("filter", function(filters) {
      updateCharts(this);
    });
    
  // Create histogram for values
  const valueChart = barChart()
    .dimension(byValue)
    .group(byValue.group(binValue))
    .x(d3.scaleLinear())
    .y(d3.scaleLinear())
    .on("filter", function(filters) {
      updateCharts(this);
    });
    
  // Create time series chart
  const dateChart = lineChart()
    .dimension(byDate)
    .group(byDate.group(d3.timeMonth))
    .x(d3.scaleTime())
    .y(d3.scaleLinear())
    .on("filter", function(filters) {
      updateCharts(this);
    });
    
  // Function to update all charts
  function updateCharts(exceptChart) {
    if (exceptChart !== categoryChart) categoryChart.update();
    if (exceptChart !== valueChart) valueChart.update();
    if (exceptChart !== dateChart) dateChart.update();
    
    // Update count display
    updateCountDisplay();
  }
  
  // Function to update count display
  function updateCountDisplay() {
    const all = cf.groupAll();
    d3.select("#count").text(all.value());
  }
  
  // Function to reset all filters
  function resetAll() {
    byCategory.filterAll();
    byValue.filterAll();
    byDate.filterAll();
    
    updateCharts();
  }
  
  // Initialize charts
  categoryChart.render();
  valueChart.render();
  dateChart.render();
  updateCountDisplay();
  
  // Return public methods and objects
  return {
    cf,
    dimensions: {
      byCategory,
      byValue,
      byDate
    },
    charts: {
      categoryChart,
      valueChart,
      dateChart
    },
    resetAll
  };
}

// 11. Coordinated Multi-View Visualization
function createCoordinatedViews(data) {
  // Set up container divs
  const container = d3.select("#dashboard")
    .style("display", "grid")
    .style("grid-template-columns", "1fr 1fr")
    .style("grid-template-rows", "auto 1fr 1fr")
    .style("gap", "20px")
    .style("height", "800px");
    
  // Add header
  container.append("div")
    .attr("id", "header")
    .style("grid-column", "1 / span 2")
    .html("<h1>Dashboard</h1>");
    
  // Add chart containers
  const scatterContainer = container.append("div")
    .attr("id", "scatter-chart")
    .attr("class", "chart-container");
    
  const barContainer = container.append("div")
    .attr("id", "bar-chart")
    .attr("class", "chart-container");
    
  const lineContainer = container.append("div")
    .attr("id", "line-chart")
    .attr("class", "chart-container");
    
  const mapContainer = container.append("div")
    .attr("id", "map-chart")
    .attr("class", "chart-container");
    
  // Create dispatch for event coordination
  const dispatch = d3.dispatch("highlight", "select", "filter");
  
  // Create scatter plot
  const scatter = createScatterPlot(scatterContainer.node(), data, dispatch);
  
  // Create bar chart
  const bar = createBarChart(barContainer.node(), data, dispatch);
  
  // Create line chart
  const line = createLineChart(lineContainer.node(), data, dispatch);
  
  // Create map
  const map = createMapChart(mapContainer.node(), data, dispatch);
  
  // Set up event handlers
  dispatch.on("highlight", id => {
    scatter.highlight(id);
    bar.highlight(id);
    line.highlight(id);
    map.highlight(id);
  });
  
  dispatch.on("select", id => {
    scatter.select(id);
    bar.select(id);
    line.select(id);
    map.select(id);
    
    // Update detail panel
    updateDetailPanel(id);
  });
  
  dispatch.on("filter", filter => {
    const filteredData = filterData(data, filter);
    scatter.update(filteredData);
    bar.update(filteredData);
    line.update(filteredData);
    map.update(filteredData);
  });
  
  // Create filter controls
  createFilterControls(data, dispatch);
  
  // Create detail panel
  createDetailPanel();
  
  // Function to update detail panel
  function updateDetailPanel(id) {
    const item = data.find(d => d.id === id);
    if (!item) return;
    
    const panel = d3.select("#detail-panel");
    
    panel.html("")
      .append("h2")
        .text(item.name)
      .append("p")
        .text(`Category: ${item.category}`)
      .append("p")
        .text(`Value: ${item.value}`)
      .append("p")
        .text(`Date: ${d3.timeFormat("%B %d, %Y")(item.date)}`);
        
    // Add more details and mini-charts as needed
  }
  
  // Return public methods and objects
  return {
    dispatch,
    charts: {
      scatter,
      bar,
      line,
      map
    },
    updateDetailPanel
  };
}

// 12. Interactive Legends
function createInteractiveLegend(colorScale, onChange) {
  // Create legend container
  const legend = d3.select("#legend")
    .append("div")
    .attr("class", "legend")
    .style("display", "flex")
    .style("flex-direction", "column")
    .style("gap", "5px");
    
  // Get domain from color scale
  const categories = colorScale.domain();
  
  // State for toggled categories
  const disabled = new Set();
  
  // Create legend items
  const items = legend.selectAll(".legend-item")
    .data(categories)
    .join("div")
      .attr("class", "legend-item")
      .style("display", "flex")
      .style("align-items", "center")
      .style("cursor", "pointer")
      .on("click", function(event, d) {
        // Toggle category
        if (disabled.has(d)) {
          disabled.delete(d);
        } else {
          disabled.add(d);
        }
        
        // Update visual appearance
        d3.select(this)
          .style("opacity", disabled.has(d) ? 0.5 : 1);
          
        // Call change handler
        if (onChange) {
          onChange(Array.from(categories).filter(c => !disabled.has(c)));
        }
      });
      
  // Add color swatches
  items.append("div")
    .attr("class", "legend-swatch")
    .style("width", "15px")
    .style("height", "15px")
    .style("margin-right", "5px")
    .style("background-color", d => colorScale(d));
    
  // Add text labels
  items.append("span")
    .text(d => d);
    
  // Add "Select All" button
  legend.append("button")
    .attr("class", "select-all")
    .text("Select All")
    .style("margin-top", "10px")
    .on("click", () => {
      disabled.clear();
      
      items.style("opacity", 1);
      
      if (onChange) {
        onChange(Array.from(categories));
      }
    });
    
  // Add "Clear All" button
  legend.append("button")
    .attr("class", "clear-all")
    .text("Clear All")
    .style("margin-top", "5px")
    .on("click", () => {
      categories.forEach(c => disabled.add(c));
      
      items.style("opacity", 0.5);
      
      if (onChange) {
        onChange([]);
      }
    });
    
  // Return public methods
  return {
    getEnabled: () => Array.from(categories).filter(c => !disabled.has(c)),
    setEnabled: (enabledCategories) => {
      disabled.clear();
      categories.forEach(c => {
        if (!enabledCategories.includes(c)) {
          disabled.add(c);
        }
      });
      
      items.style("opacity", d => disabled.has(d) ? 0.5 : 1);
    }
  };
}

// 13. Advanced SVG filter effects
function addFilterEffects(svg) {
  // Create defs for filters
  const defs = svg.append("defs");
  
  // Drop shadow filter
  const dropShadow = defs.append("filter")
    .attr("id", "drop-shadow")
    .attr("height", "130%");
    
  dropShadow.append("feGaussianBlur")
    .attr("in", "SourceAlpha")
    .attr("stdDeviation", 3)
    .attr("result", "blur");
    
  dropShadow.append("feOffset")
    .attr("in", "blur")
    .attr("dx", 2)
    .attr("dy", 2)
    .attr("result", "offsetBlur");
    
  const feMerge = dropShadow.append("feMerge");
  feMerge.append("feMergeNode")
    .attr("in", "offsetBlur");
  feMerge.append("feMergeNode")
    .attr("in", "SourceGraphic");
    
  // Glow filter
  const glow = defs.append("filter")
    .attr("id", "glow")
    .attr("x", "-50%")
    .attr("y", "-50%")
    .attr("width", "200%")
    .attr("height", "200%");
    
  glow.append("feGaussianBlur")
    .attr("stdDeviation", "3.5")
    .attr("result", "coloredBlur");
    
  const feFlood = glow.append("feFlood")
    .attr("flood-color", "#5aacff")
    .attr("flood-opacity", "0.8")
    .attr("result", "glowColor");
    
  glow.append("feComposite")
    .attr("in", "glowColor")
    .attr("in2", "coloredBlur")
    .attr("operator", "in")
    .attr("result", "softGlow_colored");
    
  const feMerge2 = glow.append("feMerge");
  feMerge2.append("feMergeNode")
    .attr("in", "softGlow_colored");
  feMerge2.append("feMergeNode")
    .attr("in", "SourceGraphic");
    
  // Inset shadow filter
  const insetShadow = defs.append("filter")
    .attr("id", "inset-shadow")
    .attr("x", "-50%")
    .attr("y", "-50%")
    .attr("width", "200%")
    .attr("height", "200%");
    
  insetShadow.append("feComponentTransfer")
    .attr("in", "SourceAlpha")
    .append("feFuncA")
    .attr("type", "table")
    .attr("tableValues", "1 0");
    
  insetShadow.append("feGaussianBlur")
    .attr("stdDeviation", "2")
    .attr("result", "blur");
    
  insetShadow.append("feOffset")
    .attr("dx", "2")
    .attr("dy", "2")
    .attr("result", "offsetBlur");
    
  insetShadow.append("feComposite")
    .attr("in", "offsetBlur")
    .attr("in2", "SourceAlpha")
    .attr("operator", "in")
    .attr("result", "innerShadow");
    
  const feMerge3 = insetShadow.append("feMerge");
  feMerge3.append("feMergeNode")
    .attr("in", "SourceGraphic");
  feMerge3.append("feMergeNode")
    .attr("in", "innerShadow");
    
  // Usage example
  svg.selectAll(".highlight-element")
    .attr("filter", "url(#glow)");
    
  svg.selectAll(".shadow-element")
    .attr("filter", "url(#drop-shadow)");
    
  svg.selectAll(".inset-element")
    .attr("filter", "url(#inset-shadow)");
}

// 14. Dynamic loading of large datasets
function loadLargeDataset(url, progressCallback, completeCallback) {
  // Create streaming parser for large CSV
  const rowCount = 0;
  const chunkSize = 10000; // Process in chunks of 10,000 rows
  let data = [];
  let headers = null;
  
  // Function to process chunks of data
  function processChunk(chunk, parser) {
    // Parse the chunk
    const result = parser.parse(chunk);
    
    // If first chunk, get headers
    if (!headers && result.data.length > 0) {
      headers = result.data[0];
      // Remove header row
      result.data.shift();
    }
    
    // Process the data rows
    data = data.concat(result.data.filter(row => row.length === headers.length));
    
    // Update progress
    if (progressCallback) {
      progressCallback({
        processed: data.length,
        total: null // Unknown until complete
      });
    }
    
    // If enough rows for a chunk, process visualization
    if (data.length >= chunkSize) {
      // Clone the current chunk
      const currentChunk = data.slice(0, chunkSize);
      
      // Update visualization with current chunk
      updateVisualization(currentChunk);
      
      // Keep the rest for next chunk
      data = data.slice(chunkSize);
    }
  }
  
  // Stream the CSV file
  fetch(url)
    .then(response => {
      // Get response body as ReadableStream
      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      let buffer = '';
      
      // Create CSV parser
      const parser = new Papa.Parser();
      
      // Process stream
      function processStream({ done, value }) {
        if (done) {
          // Process remaining data
          if (buffer.length > 0) {
            processChunk(buffer, parser);
          }
          
          // Final update with all data
          updateVisualization(data, true);
          
          // Complete callback
          if (completeCallback) {
            completeCallback({
              headers,
              data,
              total: data.length
            });
          }
          
          return;
        }
        
        // Decode chunk and add to buffer
        buffer += decoder.decode(value, { stream: true });
        
        // Split by newlines, keeping last partial line in buffer
        const lines = buffer.split(/\r\n|\n|\r/);
        buffer = lines.pop();
        
        // Process complete lines
        const chunk = lines.join('\n');
        processChunk(chunk, parser);
        
        // Continue reading
        return reader.read().then(processStream);
      }
      
      // Start reading
      return reader.read().then(processStream);
    })
    .catch(error => {
      console.error("Error loading data:", error);
      if (completeCallback) {
        completeCallback({ error });
      }
    });
    
  // Function to update visualization with current data
  function updateVisualization(currentData, isFinal = false) {
    // Implementation depends on visualization type
    console.log(`Updating visualization with ${currentData.length} rows. Final: ${isFinal}`);
    
    // Example: update scatter plot
    svg.selectAll("circle")
      .data(currentData, d => d.id)
      .join(
        enter => enter.append("circle")
          .attr("cx", d => x(d.x))
          .attr("cy", d => y(d.y))
          .attr("r", 3)
          .attr("fill", "steelblue")
          .attr("opacity", 0)
          .call(enter => enter.transition()
            .duration(500)
            .attr("opacity", 0.7)
          ),
        update => update,
        exit => exit.call(exit => exit.transition()
          .duration(500)
          .attr("opacity", 0)
          .remove()
        )
      );
  }
}

// 15. Custom controls for interactive parameters
function createInteractiveControls(params, onChange) {
  // Create controls container
  const container = d3.select("#controls")
    .append("div")
    .attr("class", "controls-container")
    .style("display", "flex")
    .style("flex-wrap", "wrap")
    .style("gap", "20px");
    
  // Function to create slider
  function createSlider(param) {
    const control = container.append("div")
      .attr("class", "control slider-control")
      .style("display", "flex")
      .style("flex-direction", "column")
      .style("min-width", "200px");
      
    control.append("label")
      .attr("for", `slider-${param.id}`)
      .text(param.label);
      
    const valueDisplay = control.append("div")
      .attr("class", "value-display")
      .text(param.value);
      
    const slider = control.append("input")
      .attr("id", `slider-${param.id}`)
      .attr("type", "range")
      .attr("min", param.min)
      .attr("max", param.max)
      .attr("step", param.step || 1)
      .attr("value", param.value)
      .on("input", function() {
        const value = parseFloat(this.value);
        valueDisplay.text(param.format ? param.format(value) : value);
        
        if (onChange) {
          onChange({
            id: param.id,
            value
          });
        }
      });
      
    return {
      getValue: () => parseFloat(slider.property("value")),
      setValue: (value) => {
        slider.property("value", value);
        valueDisplay.text(param.format ? param.format(value) : value);
      }
    };
  }
  
  // Function to create dropdown
  function createDropdown(param) {
    const control = container.append("div")
      .attr("class", "control dropdown-control")
      .style("display", "flex")
      .style("flex-direction", "column")
      .style("min-width", "200px");
      
    control.append("label")
      .attr("for", `dropdown-${param.id}`)
      .text(param.label);
      
    const select = control.append("select")
      .attr("id", `dropdown-${param.id}`)
      .on("change", function() {
        const value = this.value;
        
        if (onChange) {
          onChange({
            id: param.id,
            value
          });
        }
      });
      
    select.selectAll("option")
      .data(param.options)
      .join("option")
        .attr("value", d => d.value)
        .property("selected", d => d.value === param.value)
        .text(d => d.label);
        
    return {
      getValue: () => select.property("value"),
      setValue: (value) => {
        select.property("value", value);
      }
    };
  }
  
  // Function to create checkbox
  function createCheckbox(param) {
    const control = container.append("div")
      .attr("class", "control checkbox-control")
      .style("display", "flex")
      .style("align-items", "center")
      .style("min-width", "200px");
      
    const checkbox = control.append("input")
      .attr("id", `checkbox-${param.id}`)
      .attr("type", "checkbox")
      .property("checked", param.value)
      .style("margin-right", "8px")
      .on("change", function() {
        const value = this.checked;
        
        if (onChange) {
          onChange({
            id: param.id,
            value
          });
        }
      });
      
    control.append("label")
      .attr("for", `checkbox-${param.id}`)
      .text(param.label);
      
    return {
      getValue: () => checkbox.property("checked"),
      setValue: (value) => {
        checkbox.property("checked", value);
      }
    };
  }
  
  // Function to create radio group
  function createRadioGroup(param) {
    const control = container.append("div")
      .attr("class", "control radio-control")
      .style("display", "flex")
      .style("flex-direction", "column")
      .style("min-width", "200px");
      
    control.append("span")
      .attr("class", "radio-group-label")
      .text(param.label);
      
    const radioGroup = control.append("div")
      .attr("class", "radio-options")
      .style("display", "flex")
      .style("flex-direction", "column")
      .style("margin-top", "5px");
      
    const radios = radioGroup.selectAll(".radio-option")
      .data(param.options)
      .join("div")
        .attr("class", "radio-option")
        .style("display", "flex")
        .style("align-items", "center")
        .style("margin-bottom", "5px");
        
    radios.append("input")
      .attr("type", "radio")
      .attr("name", `radio-${param.id}`)
      .attr("id", (d, i) => `radio-${param.id}-${i}`)
      .attr("value", d => d.value)
      .property("checked", d => d.value === param.value)
      .style("margin-right", "8px")
      .on("change", function(event, d) {
        if (this.checked && onChange) {
          onChange({
            id: param.id,
            value: d.value
          });
        }
      });
      
    radios.append("label")
      .attr("for", (d, i) => `radio-${param.id}-${i}`)
      .text(d => d.label);
      
    return {
      getValue: () => {
        const checked = control.select("input:checked");
        return checked.empty() ? null : checked.property("value");
      },
      setValue: (value) => {
        control.selectAll("input")
          .property("checked", d => d.value === value);
      }
    };
  }
  
  // Function to create color picker
  function createColorPicker(param) {
    const control = container.append("div")
      .attr("class", "control color-control")
      .style("display", "flex")
      .style("flex-direction", "column")
      .style("min-width", "200px");
      
    control.append("label")
      .attr("for", `color-${param.id}`)
      .text(param.label);
      
    const picker = control.append("input")
      .attr("id", `color-${param.id}`)
      .attr("type", "color")
      .attr("value", param.value)
      .on("input", function() {
        const value = this.value;
        
        if (onChange) {
          onChange({
            id: param.id,
            value
          });
        }
      });
      
    return {
      getValue: () => picker.property("value"),
      setValue: (value) => {
        picker.property("value", value);
      }
    };
  }
  
  // Create controls based on parameter types
  const controls = {};
  
  params.forEach(param => {
    let control;
    
    switch (param.type) {
      case "slider":
        control = createSlider(param);
        break;
      case "dropdown":
        control = createDropdown(param);
        break;
      case "checkbox":
        control = createCheckbox(param);
        break;
      case "radio":
        control = createRadioGroup(param);
        break;
      case "color":
        control = createColorPicker(param);
        break;
      default:
        console.warn(`Unknown parameter type: ${param.type}`);
        return;
    }
    
    controls[param.id] = control;
  });
  
  // Add reset button
  container.append("button")
    .attr("class", "reset-button")
    .text("Reset to Defaults")
    .style("margin-top", "20px")
    .on("click", () => {
      // Reset all controls to default values
      params.forEach(param => {
        if (controls[param.id]) {
          controls[param.id].setValue(param.defaultValue);
        }
        
        if (onChange) {
          onChange({
            id: param.id,
            value: param.defaultValue
          });
        }
      });
    });
    
  // Return controls object
  return controls;
}
```## Best Practices and Advanced Techniques

### Performance Optimization
```javascript
// 1. Use SVG clipping paths for large datasets
svg.append("defs").append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
    .attr("width", width)
    .attr("height", height);

chart.attr("clip-path", "url(#chart-area)");

// 2. Use canvas for very large datasets (thousands of elements)
const canvas = d3.select("#chart")
  .append("canvas")
  .attr("width", width)
  .attr("height", height);
  
const context = canvas.node().getContext("2d");

// Draw data points on canvas
data.forEach(d => {
  context.fillStyle = colorScale(d.category);
  context.beginPath();
  context.arc(x(d.x), y(d.y), 3, 0, 2 * Math.PI);
  context.fill();
});

// 3. Use requestAnimationFrame for smooth animations
function animate() {
  // Update data
  data = updateData(data);
  
  // Update elements
  selection.data(data)
    .attr("cx", d => x(d.x))
    .attr("cy", d => y(d.y));
    
  // Request next frame
  requestAnimationFrame(animate);
}

// Start animation
requestAnimationFrame(animate);

// 4. Data filtering and decimation
// Reduce points when zoomed out
function filterData(data, zoom) {
  const factor = Math.ceil(data.length / (width / zoom.k / 5));
  return factor <= 1 ? data : data.filter((d, i) => i % factor === 0);
}

// 5. Efficient update patterns
function efficientUpdate(data) {
  // Only select/modify what needs to change
  // Bad: selectAll inside loop
  // for (let i = 0; i < data.length; i++) {
  //   d3.selectAll("circle").filter((d, j) => j === i)
  //     .attr("r", data[i].radius);
  // }
  
  // Good: Set data once, update all
  const circles = svg.selectAll("circle")
    .data(data);
    
  circles.attr("r", d => d.radius);
}

// 6. Throttle/debounce event handlers
function throttle(callback, delay) {
  let last = 0;
  return function(...args) {
    const now = Date.now();
    if (now - last >= delay) {
      callback.apply(this, args);
      last = now;
    }
  };
}

svg.call(zoom)
  .on("wheel", throttle(function(event) {
    // Handle zoom
  }, 50));
```

### Reusable Components
```javascript
// Create a reusable bar chart component
function barChart() {
  // Default values
  let width = 600;
  let height = 400;
  let margin = {top: 20, right: 20, bottom: 30, left: 40};
  let xValue = d => d.x;
  let yValue = d => d.y;
  let color = "steelblue";
  let duration = 500;
  
  // Create chart function
  function chart(selection) {
    selection.each(function(data) {
      // Create or select SVG element
      const svg = d3.select(this).selectAll("svg")
        .data([data])
        .join("svg")
          .attr("width", width)
          .attr("height", height)
          .attr("viewBox", `0 0 ${width} ${height}`)
          .attr("style", "max-width: 100%; height: auto;");
          
      // Create or select container group
      const g = svg.selectAll(".container")
        .data([data])
        .join("g")
          .attr("class", "container")
          .attr("transform", `translate(${margin.left},${margin.top})`);
          
      // Calculate inner dimensions
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;
      
      // Create scales
      const x = d3.scaleBand()
        .domain(data.map(xValue))
        .range([0, innerWidth])
        .padding(0.1);
        
      const y = d3.scaleLinear()
        .domain([0, d3.max(data, yValue)])
        .nice()
        .range([innerHeight, 0]);
        
      // Create or update axes
      const xAxis = g.selectAll(".x-axis")
        .data([data])
        .join("g")
          .attr("class", "x-axis")
          .attr("transform", `translate(0,${innerHeight})`)
          .transition()
          .duration(duration)
          .call(d3.axisBottom(x));
          
      const yAxis = g.selectAll(".y-axis")
        .data([data])
        .join("g")
          .attr("class", "y-axis")
          .transition()
          .duration(duration)
          .call(d3.axisLeft(y));
          
      // Create or update bars
      const bars = g.selectAll(".bar")
        .data(data)
        .join(
          enter => enter.append("rect")
            .attr("class", "bar")
            .attr("x", d => x(xValue(d)))
            .attr("width", x.bandwidth())
            .attr("y", innerHeight)
            .attr("height", 0)
            .attr("fill", color)
            .call(enter => enter.transition()
              .duration(duration)
              .attr("y", d => y(yValue(d)))
              .attr("height", d => innerHeight - y(yValue(d)))
            ),
          update => update.transition()
            .duration(duration)
            .attr("x", d => x(xValue(d)))
            .attr("width", x.bandwidth())
            .attr("y", d => y(yValue(d)))
            .attr("height", d => innerHeight - y(yValue(d))),
          exit => exit.transition()
            .duration(duration)
            .attr("y", innerHeight)
            .attr("height", 0)
            .remove()
        );
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
  
  chart.margin = function(value) {
    if (!arguments.length) return margin;
    margin = {...margin, ...value};
    return chart;
  };
  
  chart.x = function(value) {
    if (!arguments.length) return xValue;
    xValue = value;
    return chart;
  };
  
  chart.y = function(value) {
    if (!arguments.length) return yValue;
    yValue = value;
    return chart;
  };
  
  chart.color = function(value) {
    if (!arguments.length) return color;
    color = value;
    return chart;
  };
  
  chart.duration = function(value) {
    if (!arguments.length) return duration;
    duration = value;
    return chart;
  };
  
  return chart;
}

// Usage:
const myChart = barChart()
  .width(800)
  .height(500)
  .margin({top: 30, right: 30, bottom: 40, left: 50})
  .x(d => d.name)
  .y(d => d.value)
  .color("#4682b4")
  .duration(750);
  
d3.select("#chart")
  .datum(data)
  .call(myChart);
```

### Advanced Data Visualization Patterns
```javascript
// 1. Observable Pattern for Linked Views
function createLinkedViews() {
  // Create a dispatch to handle events
  const dispatch = d3.dispatch("highlight", "select", "filter");
  
  // Create charts
  const scatterplot = createScatterplot()
    .on("highlight", id => {
      dispatch.call("highlight", null, id);
    })
    .on("select", id => {
      dispatch.call("select", null, id);
    });
    
  const histogram = createHistogram()
    .on("filter", range => {
      dispatch.call("filter", null, range);
    });
    
  // Set up event listeners
  dispatch.on("highlight", id => {
    scatterplot.highlight(id);
    histogram.highlight(id);
  });
  
  dispatch.on("select", id => {
    scatterplot.select(id);
    histogram.select(id);
  });
  
  dispatch.on("filter", range => {
    scatterplot.filter(range);
  });
  
  return {
    scatterplot,
    histogram,
    dispatch
  };
}

// 2. Data Joins for Complex Visualizations
function createComplexViz(data) {
  // Group data
  const nestedData = d3.group(data, d => d.category);
  
  // First-level join - categories
  const categories = svg.selectAll(".category")
    .data(Array.from(nestedData.entries()))
    .join("g")
      .attr("class", d => `category category-${d[0]}`)
      .attr("transform", (d, i) => `translate(0,${i * 100})`);
      
  // Add category labels
  categories.selectAll(".category-label")
    .data(d => [d])
    .join("text")
      .attr("class", "category-label")
      .attr("x", 0)
      .attr("y", -10)
      .text(d => d[0]);
      
  // Second-level join - items within categories
  const items = categories.selectAll(".item")
    .data(d => d[1].map(item => ({
      category: d[0],
      ...item
    })))
    .join("g")
      .attr("class", "item")
      .attr("transform", (d, i) => `translate(${i * 30},0)`);
      
  // Add item circles
  items.selectAll("circle")
    .data(d => [d])
    .join("circle")
      .attr("r", d => sizeScale(d.value))
      .attr("fill", d => colorScale(d.type));
      
  // Add item labels
  items.selectAll(".item-label")
    .data(d => [d])
    .join("text")
      .attr("class", "item-label")
      .attr("text-anchor", "middle")
      .attr("dy", d => sizeScale(d.value) + 12)
      .text(d => d.id);
}

// 3. Advanced Tooltips
function createAdvancedTooltip() {
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ddd")
    .style("border-radius", "3px")
    .style("padding", "10px")
    .style("box-shadow", "0 2px 4px rgba(0,0,0,0.1)")
    .style("pointer-events", "none")
    .style("font-family", "sans-serif")
    .style("font-size", "12px");
    
  // Add tooltip header
  tooltip.append("div")
    .attr("class", "tooltip-header")
    .style("font-weight", "bold")
    .style("margin-bottom", "5px");
    
  // Add tooltip content
  tooltip.append("div")
    .attr("class", "tooltip-content");
    
  // Add small chart inside tooltip
  const chartWidth = 150;
  const chartHeight = 50;
  
  const tooltipSvg = tooltip.append("svg")
    .attr("width", chartWidth)
    .attr("height", chartHeight)
    .style("margin-top", "5px");
    
  const tooltipX = d3.scaleLinear()
    .range([0, chartWidth]);
    
  const tooltipY = d3.scaleLinear()
    .range([chartHeight, 0]);
    
  const tooltipLine = d3.line()
    .x((d, i) => tooltipX(i))
    .y(d => tooltipY(d));
    
  const tooltipPath = tooltipSvg.append("path")
    .attr("fill", "none")
    .attr("stroke", "#69b3a2")
    .attr("stroke-width", 1.5);
    
  // Tooltip show function
  function show(event, d) {
    // Position tooltip
    tooltip
      .style("visibility", "visible")
      .style("left", `${event.pageX + 10}px`)
      .style("top", `${event.pageY - 10}px`);
      
    // Update tooltip content
    tooltip.select(".tooltip-header")
      .text(`${d.name} (${d.category})`);
      
    tooltip.select(".tooltip-content")
      .html(`Value: <strong>${d.value}</strong><br>
             Rank: <strong>${d.rank}</strong>`);
             
    // Update tooltip chart
    if (d.history) {
      tooltipX.domain([0, d.history.length - 1]);
      tooltipY.domain([0, d3.max(d.history)]).nice();
      
      tooltipPath.datum(d.history)
        .attr("d", tooltipLine);
    } else {
      tooltipSvg.style("display", "none");
    }
  }
  
  // Tooltip move function
  function move(event) {
    tooltip
      .style("left", `${event.pageX + 10}px`)
      .style("top", `${event.pageY - 10}px`);
  }
  
  // Tooltip hide function
  function hide() {
    tooltip.style("visibility", "hidden");
  }
  
  return {
    show,
    move,
    hide
  };
}

// 4. Time-series with focus+context (brush and zoom)
function createTimeseriesWithFocus(data) {
  // Set up dimensions
  const margin = {top: 20, right: 20, bottom: 100, left: 50};
  const margin2 = {top: 430, right: 20, bottom: 30, left: 50};
  const width = 960 - margin.left - margin.right;
  const height = 500 - margin.top - margin.bottom;
  const height2 = 500 - margin2.top - margin2.bottom;
  
  // Create scales
  const x = d3.scaleTime()
    .domain(d3.extent(data, d => d.date))
    .range([0, width]);
    
  const x2 = d3.scaleTime()
    .domain(x.domain())
    .range([0, width]);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .nice()
    .range([height, 0]);
    
  const y2 = d3.scaleLinear()
    .domain(y.domain())
    .range([height2, 0]);
    
  // Create line generators
  const line = d3.line()
    .x(d => x(d.date))
    .y(d => y(d.value));
    
  const line2 = d3.line()
    .x(d => x2(d.date))
    .y(d => y2(d.value));
    
  // Create SVG
  const svg = d3.select("#chart").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom);
    
  // Clip path for main chart
  svg.append("defs").append("clipPath")
    .attr("id", "clip")
    .append("rect")
      .attr("width", width)
      .attr("height", height);
      
  // Main chart group
  const focus = svg.append("g")
    .attr("class", "focus")
    .attr("transform", `translate(${margin.left},${margin.top})`);
    
  // Context (small) chart group
  const context = svg.append("g")
    .attr("class", "context")
    .attr("transform", `translate(${margin2.left},${margin2.top})`);
    
  // Add main chart path
  focus.append("path")
    .datum(data)
    .attr("class", "line")
    .attr("clip-path", "url(#clip)")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1.5)
    .attr("d", line);
    
  // Add main chart axes
  focus.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  focus.append("g")
    .attr("class", "axis axis--y")
    .call(d3.axisLeft(y));
    
  // Add context chart path
  context.append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1)
    .attr("d", line2);
    
  // Add context chart axis
  context.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height2})`)
    .call(d3.axisBottom(x2));
    
  // Add brush
  const brush = d3.brushX()
    .extent([[0, 0], [width, height2]])
    .on("brush end", brushed);
    
  context.append("g")
    .attr("class", "brush")
    .call(brush)
    .call(brush.move, x.range());
    
  // Add zoom
  const zoom = d3.zoom()
    .scaleExtent([1, Infinity])
    .translateExtent([[0, 0], [width, height]])
    .extent([[0, 0], [width, height]])
    .on("zoom", zoomed);
    
  svg.append("rect")
    .attr("class", "zoom")
    .attr("width", width)
    .attr("height", height)
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .attr("fill", "none")
    .attr("pointer-events", "all")
    .call(zoom);
    
  // Brush handler
  function brushed(event) {
    if (event.sourceEvent && event.sourceEvent.type === "zoom") return;
    
    const selection = event.selection || x2.range();
    x.domain(selection.map(x2.invert, x2));
    
    focus.select(".line")
      .attr("d", line);
      
    focus.select(".axis--x")
      .call(d3.axisBottom(x));
      
    svg.select(".zoom")
      .call(zoom.transform, d3.zoomIdentity
        .scale(width /## D3.js Version Differences

### Version Migration Guide

```javascript
// D3.js has gone through several major versions with breaking changes
// Here's a quick reference for migrating between versions

// D3 v7 (Latest as of 2025)
// -------------------------------------
// Key changes:
// - Improved TypeScript support
// - Use of native browser APIs like ResizeObserver
// - Enhanced internationalization
// - Modernized modules and ES imports
// - Standardized use of Fetch API

// Load data using promises and fetch API
d3.json("data.json").then(data => {
  console.log(data);
}).catch(error => {
  console.error("Error loading data: ", error);
});

// Import specific modules
import {select, selectAll} from "d3-selection";
import {scaleLinear, scaleBand} from "d3-scale";

// Event handling with separate event object
selection.on("click", (event, d) => {
  console.log("Clicked:", d);
  console.log("Event:", event);
});

// D3 v6
// -------------------------------------
// Key changes:
// - Removed d3.event global (replaced with event in callback)
// - Dropped support for Internet Explorer
// - Consolidation of APIs and functions

// Event handling with separate event object
selection.on("click", (event, d) => {
  console.log("Clicked:", d);
  console.log("Event:", event);
});

// Internal properties have _ prefix (internal implementation)
const scale = d3.scaleLinear();
// accessing internal variables (not recommended)
console.log(scale._domain); 

// D3 v5
// -------------------------------------
// Key changes:
// - Promises instead of callbacks
// - Introduced d3.group and d3.rollup (replacing d3.nest)
// - Collection.join() for enter/update/exit
// - Updated TopoJSON to 3.0

// Promises instead of callbacks
d3.json("data.json").then(data => {
  console.log(data);
}).catch(error => {
  console.error("Error loading data:", error);
});

// Before (v4):
d3.json("data.json", function(error, data) {
  if (error) throw error;
  console.log(data);
});

// Join pattern for enter/update/exit
selection.join("circle"); // Simple join
selection.join(
  enter => enter.append("circle").attr("fill", "green"),
  update => update.attr("fill", "black"),
  exit => exit.attr("fill", "red").remove()
); // Full join with callbacks

// Data grouping (replacing nest)
// Before (v4):
const nested = d3.nest()
  .key(d => d.category)
  .entries(data);

// After (v5+):
const grouped = d3.group(data, d => d.category);

// D3 v4
// -------------------------------------
// Key changes:
// - Modular structure (no more d3.scale, use d3.scaleLinear etc.)
// - Named exports
// - d3.schemeCategory20 replaced with d3.schemeCategory10
// - Simpler force layouts
// - Mouse events normalized across browsers

// Modular structure
// Before (v3):
const x = d3.scale.linear();

// After (v4+):
const x = d3.scaleLinear();

// Named exports
import {select} from "d3-selection";
import {scaleLinear} from "d3-scale";

// Force layout changes
// Before (v3):
const force = d3.layout.force()
  .nodes(nodes)
  .links(links)
  .size([width, height])
  .start();

// After (v4+):
const simulation = d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links))
  .force("charge", d3.forceManyBody())
  .force("center", d3.forceCenter(width / 2, height / 2));

// D3 v3
// -------------------------------------
// This was the last version with a monolithic structure
// Still widely used in legacy code and examples

// Global selection methods
d3.select("#chart");
d3.selectAll(".item");

// Chained methods
d3.select("#chart")
  .append("svg")
  .attr("width", 500)
  .attr("height", 300);

// Scales
const x = d3.scale.linear()
  .domain([0, 100])
  .range([0, width]);

// Force layout
const force = d3.layout.force()
  .size([width, height])
  .nodes(nodes)
  .links(links);

// Event handling using d3.event
selection.on("click", function(d) {
  console.log("Clicked:", d);
  console.log("Event:", d3.event);
});

// Data loading with callbacks
d3.json("data.json", function(error, data) {
  if (error) throw error;
  console.log(data);
});

// Notable Breaking Changes between Versions
// =========================================

// 1. Event Handling
// v3: Uses global d3.event
selection.on("click", function(d) {
  const mouse = d3.mouse(this); // [x, y] relative to this element
  console.log(d3.event.type);   // "click"
});

// v4-v5: Still uses global d3.event but with better compatibility
selection.on("click", function(d) {
  const mouse = d3.mouse(this);
  console.log(d3.event.type);
});

// v6+: Event passed as first parameter (no global d3.event)
selection.on("click", function(event, d) {
  const mouse = d3.pointer(event, this);
  console.log(event.type);
});

// 2. Scales
// v3: Namespace under d3.scale
const linear = d3.scale.linear();
const ordinal = d3.scale.ordinal();
const log = d3.scale.log();

// v4+: Direct functions
const linear = d3.scaleLinear();
const ordinal = d3.scaleOrdinal();
const log = d3.scaleLog();

// 3. Color Schemes
// v3: Limited built-in color schemes
const color = d3.scale.category10(); // 10 colors
const color20 = d3.scale.category20(); // 20 colors

// v4-v5: Introduced d3-scale-chromatic
const color = d3.scaleOrdinal(d3.schemeCategory10);
// d3.schemeCategory20 removed in v5

// v5+: More schemes in d3-scale-chromatic
const color = d3.scaleOrdinal(d3.schemeCategory10);
const blues = d3.scaleSequential(d3.interpolateBlues);

// 4. Hierarchical Layouts
// v3: Uses specific namespace
const tree = d3.layout.tree();
const pack = d3.layout.pack();

// v4+: Simplified API with d3-hierarchy
const root = d3.hierarchy(data);
const tree = d3.tree();
tree(root); // Apply layout to hierarchy

// 5. Force Layout
// v3: Single configuration
const force = d3.layout.force()
  .charge(-300)
  .linkDistance(50)
  .size([width, height])
  .on("tick", tick);

// v4+: Modular forces
const simulation = d3.forceSimulation()
  .force("charge", d3.forceManyBody().strength(-300))
  .force("link", d3.forceLink().distance(50))
  .force("center", d3.forceCenter(width/2, height/2))
  .on("tick", tick);

// 6. Axis
// v3: Scale passed as parameter
const xAxis = d3.svg.axis()
  .scale(x)
  .orient("bottom");

// v4+: Scale determines axis type
const xAxis = d3.axisBottom(x);

// 7. Line Generators
// v3: Separate x and y accessors
const line = d3.svg.line()
  .x(function(d) { return x(d.date); })
  .y(function(d) { return y(d.value); });

// v4+: Similar but under d3.line
const line = d3.line()
  .x(d => x(d.date))
  .y(d => y(d.value));

// 8. JSON Loading
// v3: Callbacks
d3.json("data.json", function(error, data) {
  if (error) throw error;
  console.log(data);
});

// v5+: Promises
d3.json("data.json").then(data => {
  console.log(data);
}).catch(error => {
  console.error(error);
});

// v6+: Full Fetch API compatibility
d3.json("data.json", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ query: "value" })
}).then(data => {
  console.log(data);
});

// 9. Selections
// v3-v4: Enter/Update/Exit pattern
const update = selection.data(data);
const enter = update.enter().append("circle");
update.exit().remove();
update.merge(enter)
  .attr("r", d => d.radius);

// v5+: Join pattern
selection.data(data)
  .join("circle")
  .attr("r", d => d.radius);

// 10. Array Utilities
// v3-v4: d3.min, d3.max, etc.
const maxValue = d3.max(data, d => d.value);

// v5+: Added d3.group and d3.rollup (replacing d3.nest)
const grouped = d3.group(data, d => d.category);
const counts = d3.rollup(data, 
  v => v.length, 
  d => d.category
);

// Tips for Migration
// =================
// 1. Start with updating the loading mechanism (callbacks to promises)
// 2. Fix event handling (d3.event to event parameter)
// 3. Update scale constructors (d3.scale.linear to d3.scaleLinear)
// 4. Modify selection patterns (use .join() where possible)
// 5. Update force layouts (modular forces)
// 6. Check color schemes (use d3-scale-chromatic)
// 7. Test on all target browsers
```### Choropleth Map
```javascript
// Sample data for US states
const statesData = [
  {id: "AL", state: "Alabama", value: 45},
  {id: "AK", state: "Alaska", value: 32},
  {id: "AZ", state: "Arizona", value: 67},
  {id: "AR", state: "Arkansas", value: 28},
  // Add more states here...
  {id: "WY", state: "Wyoming", value: 41}
];

// Set up dimensions
const width = 960;
const height = 600;
const margin = {top: 40, right: 20, bottom: 40, left: 20};

// Create SVG container
const svg = d3.select("#chart")
  .append("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("style", "max-width: 100%; height: auto;");

// Add title
svg.append("text")
  .attr("x", width / 2)
  .attr("y", margin.top / 2)
  .attr("text-anchor", "middle")
  .attr("font-size", "20px")
  .attr("font-weight", "bold")
  .text("US States Choropleth Map");

// Create color scale
const colorScale = d3.scaleQuantize()
  .domain([0, 100])
  .range(d3.schemeBlues[9]);

// Create tooltip
const tooltip = d3.select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("background", "white")
  .style("border", "1px solid #ddd")
  .style("border-radius", "3px")
  .style("padding", "8px")
  .style("pointer-events", "none")
  .style("opacity", 0);

// Create map container
const mapGroup = svg.append("g")
  .attr("transform", `translate(${margin.left}, ${margin.top})`);

// Load US TopoJSON data
// This requires the topojson library: https://github.com/topojson/topojson
// You would normally load this from a file using d3.json()
// For brevity, we're assuming it's already loaded as 'us'
const path = d3.geoPath();

// The real implementation would look like:
// d3.json("https://cdn.jsdelivr.net/npm/us-atlas@3/states-albers-10m.json")
//   .then(us => {
//     // Create map with the loaded data
//     renderMap(us);
//   });

function renderMap(us) {
  // Extract state features from topojson
  const states = topojson.feature(us, us.objects.states).features;
  
  // Create lookup object for state data
  const stateDataById = {};
  statesData.forEach(d => {
    stateDataById[d.id] = d;
  });
  
  // Add state paths
  mapGroup.selectAll("path")
    .data(states)
    .join("path")
      .attr("class", "state")
      .attr("d", path)
      .attr("fill", d => {
        const stateData = stateDataById[d.id];
        return stateData ? colorScale(stateData.value) : "#ccc";
      })
      .attr("stroke", "#fff")
      .attr("stroke-width", 0.5)
      .on("mouseover", function(event, d) {
        const stateData = stateDataById[d.id];
        if (!stateData) return;
        
        // Highlight state
        d3.select(this)
          .attr("stroke", "#333")
          .attr("stroke-width", 1.5);
          
        // Show tooltip
        tooltip
          .transition()
          .duration(200)
          .style("opacity", 0.9);
          
        tooltip
          .html(`<strong>${stateData.state}</strong><br>Value: ${stateData.value}`)
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", function() {
        // Reset state styling
        d3.select(this)
          .attr("stroke", "#fff")
          .attr("stroke-width", 0.5);
          
        // Hide tooltip
        tooltip
          .transition()
          .duration(500)
          .style("opacity", 0);
      });
    
  // Add state labels
  mapGroup.selectAll("text")
    .data(states)
    .join("text")
      .attr("transform", d => `translate(${path.centroid(d)})`)
      .attr("text-anchor", "middle")
      .attr("font-size", "10px")
      .attr("pointer-events", "none")
      .text(d => {
        const stateData = stateDataById[d.id];
        return stateData ? stateData.id : "";
      });
}

// Add legend
function createLegend() {
  const legendWidth = 300;
  const legendHeight = 40;
  const legendMargin = {top: 20, right: 20, bottom: 0, left: 20};
  
  // Create legend container
  const legend = svg.append("g")
    .attr("class", "legend")
    .attr("transform", `translate(${(width - legendWidth) / 2}, ${height - legendHeight - legendMargin.bottom})`);
    
  // Create gradient for legend
  const defs = svg.append("defs");
  const linearGradient = defs.append("linearGradient")
    .attr("id", "choropleth-gradient")
    .attr("x1", "0%")
    .attr("x2", "100%")
    .attr("y1", "0%")
    .attr("y2", "0%");
    
  // Define gradient stops
  d3.schemeBlues[9].forEach((color, i) => {
    linearGradient.append("stop")
      .attr("offset", `${i * 100 / 8}%`)
      .attr("stop-color", color);
  });
  
  // Add gradient rectangle
  legend.append("rect")
    .attr("width", legendWidth)
    .attr("height", legendHeight / 2)
    .style("fill", "url(#choropleth-gradient)");
    
  // Add legend scale
  const legendScale = d3.scaleLinear()
    .domain([0, 100])
    .range([0, legendWidth]);
    
  const legendAxis = d3.axisBottom(legendScale)
    .ticks(9)
    .tickSize(6);
    
  legend.append("g")
    .attr("transform", `translate(0, ${legendHeight / 2})`)
    .call(legendAxis);
    
  // Add legend title
  legend.append("text")
    .attr("x", legendWidth / 2)
    .attr("y", -5)
    .attr("text-anchor", "middle")
    .attr("font-size", "12px")
    .text("Value Scale");
}

// Call the legend function
createLegend();

// Animation and interactivity enhancements

// 1. Zoom and pan capability
function addZoomPan() {
  const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .on("zoom", event => {
      mapGroup.attr("transform", event.transform);
      
      // Adjust stroke width based on zoom level
      mapGroup.selectAll("path.state")
        .attr("stroke-width", 0.5 / event.transform.k);
        
      // Adjust label size based on zoom level
      mapGroup.selectAll("text")
        .attr("font-size", `${10 / event.transform.k}px`);
    });
    
  svg.call(zoom);
}

// 2. Create animated transition when data changes
function updateMapData(newData) {
  // Update the data lookup
  const stateDataById = {};
  newData.forEach(d => {
    stateDataById[d.id] = d;
  });
  
  // Update color scale if needed
  const maxValue = d3.max(newData, d => d.value);
  colorScale.domain([0, maxValue]);
  
  // Transition map colors
  mapGroup.selectAll("path.state")
    .transition()
    .duration(750)
    .attr("fill", d => {
      const stateData = stateDataById[d.id];
      return stateData ? colorScale(stateData.value) : "#ccc";
    });
    
  // Update the legend if needed
  updateLegend(0, maxValue);
}

function updateLegend(min, max) {
  // Update legend scale
  const legendScale = d3.scaleLinear()
    .domain([min, max])
    .range([0, 300]);
    
  const legendAxis = d3.axisBottom(legendScale)
    .ticks(9)
    .tickSize(6);
    
  // Transition legend
  svg.select(".legend g")
    .transition()
    .duration(750)
    .call(legendAxis);
}

// 3. Alternate color schemes
function changeColorScheme(scheme) {
  // Available schemes:
  // d3.schemeBlues, d3.schemeGreens, d3.schemeGreys, d3.schemeOranges, 
  // d3.schemePurples, d3.schemeReds, d3.schemeBuGn, d3.schemeBuPu,
  // d3.schemeGnBu, d3.schemeOrRd, d3.schemePuBu, d3.schemePuBuGn,
  // d3.schemePuRd, d3.schemeRdPu, d3.schemeYlGn, d3.schemeYlGnBu,
  // d3.schemeYlOrBr, d3.schemeYlOrRd
  
  let colorRange;
  switch(scheme) {
    case 'blues': colorRange = d3.schemeBlues[9]; break;
    case 'reds': colorRange = d3.schemeReds[9]; break;
    case 'greens': colorRange = d3.schemeGreens[9]; break;
    case 'purples': colorRange = d3.schemePurples[9]; break;
    case 'oranges': colorRange = d3.schemeOranges[9]; break;
    case 'spectral': colorRange = d3.schemeSpectral[9]; break;
    case 'viridis': colorRange = d3.interpolateViridis; break;
    default: colorRange = d3.schemeBlues[9];
  }
  
  // Update color scale
  colorScale.range(colorRange);
  
  // Update map
  mapGroup.selectAll("path.state")
    .transition()
    .duration(750)
    .attr("fill", d => {
      const stateData = stateDataById[d.id];
      return stateData ? colorScale(stateData.value) : "#ccc";
    });
    
  // Update legend gradient
  const linearGradient = svg.select("#choropleth-gradient");
  linearGradient.selectAll("stop").remove();
  
  if (Array.isArray(colorRange)) {
    // For discrete color schemes
    colorRange.forEach((color, i) => {
      linearGradient.append("stop")
        .attr("offset", `${i * 100 / (colorRange.length - 1)}%`)
        .attr("stop-color", color);
    });
  } else {
    // For interpolated color schemes
    for (let i = 0; i <= 10; i++) {
      linearGradient.append("stop")
        .attr("offset", `${i * 10}%`)
        .attr("stop-color", colorRange(i / 10));
    }
  }
}

// 4. Different map projections
function changeProjection(projectionType) {
  let projection;
  
  switch(projectionType) {
    case 'albersUsa':
      projection = d3.geoAlbersUsa().fitSize([width, height], topojson.feature(us, us.objects.states));
      break;
    case 'mercator':
      projection = d3.geoMercator().fitSize([width, height], topojson.feature(us, us.objects.states));
      break;
    case 'equalArea':
      projection = d3.geoEqualEarth().fitSize([width, height], topojson.feature(us, us.objects.states));
      break;
    case 'orthographic':
      projection = d3.geoOrthographic().fitSize([width, height], topojson.feature(us, us.objects.states));
      break;
    default:
      projection = d3.geoAlbersUsa().fitSize([width, height], topojson.feature(us, us.objects.states));
  }
  
  // Update path generator
  path = d3.geoPath().projection(projection);
  
  // Update map paths
  mapGroup.selectAll("path.state")
    .transition()
    .duration(1000)
    .attr("d", path);
    
  // Update labels
  mapGroup.selectAll("text")
    .transition()
    .duration(1000)
    .attr("transform", d => `translate(${path.centroid(d)})`);
}

// 5. Adding data drill-down for state details
function addDrillDown() {
  // When a state is clicked, zoom to it and show detailed info
  mapGroup.selectAll("path.state")
    .on("click", function(event, d) {
      const stateData = stateDataById[d.id];
      if (!stateData) return;
      
      // Get state's bounding box
      const bounds = path.bounds(d);
      const dx = bounds[1][0] - bounds[0][0];
      const dy = bounds[1][1] - bounds[0][1];
      const x = (bounds[0][0] + bounds[1][0]) / 2;
      const y = (bounds[0][1] + bounds[1][1]) / 2;
      const scale = 0.8 / Math.max(dx / width, dy / height);
      const translate = [width / 2 - scale * x, height / 2 - scale * y];
      
      // Zoom to state
      svg.transition()
        .duration(750)
        .call(
          zoom.transform,
          d3.zoomIdentity
            .translate(translate[0], translate[1])
            .scale(scale)
        );
        
      // Show detailed panel
      showDetailPanel(stateData);
    });
}

function showDetailPanel(stateData) {
  // Create or update detail panel
  let detailPanel = d3.select("#detail-panel");
  
  if (detailPanel.empty()) {
    detailPanel = d3.select("body")
      .append("div")
      .attr("id", "detail-panel")
      .style("position", "absolute")
      .style("top", "20px")
      .style("right", "20px")
      .style("width", "250px")
      .style("background", "white")
      .style("border", "1px solid #ddd")
      .style("border-radius", "5px")
      .style("padding", "15px")
      .style("box-shadow", "0 2px 5px rgba(0,0,0,0.1)");
      
    // Add close button
    detailPanel.append("button")
      .text("×")
      .style("position", "absolute")
      .style("top", "5px")
      .style("right", "10px")
      .style("border", "none")
      .style("background", "none")
      .style("font-size", "20px")
      .style("cursor", "pointer")
      .on("click", () => {
        // Close panel and reset zoom
        detailPanel.style("display", "none");
        
        svg.transition()
          .duration(750)
          .call(
            zoom.transform,
            d3.zoomIdentity
          );
      });
  }
  
  // Update panel content
  detailPanel.style("display", "block")
    .html(`
      <h3>${stateData.state} (${stateData.id})</h3>
      <hr>
      <p><strong>Value:</strong> ${stateData.value}</p>
      <!-- Add more details here -->
      <p><strong>Population:</strong> ${Math.floor(Math.random() * 10000000)}</p>
      <p><strong>Area:</strong> ${Math.floor(Math.random() * 200000)} sq mi</p>
      <div id="state-detail-chart"></div>
    `);
    
  // Add a small chart for the state
  const chartWidth = 220;
  const chartHeight = 100;
  
  const chartSvg = d3.select("#state-detail-chart")
    .append("svg")
    .attr("width", chartWidth)
    .attr("height", chartHeight);
    
  // Create fake time series data
  const fakeData = Array.from({length: 12}, (_, i) => ({
    month: i + 1,
    value: Math.random() * stateData.value * 1.5
  }));
  
  // Create scales
  const x = d3.scaleLinear()
    .domain([1, 12])
    .range([10, chartWidth - 10]);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(fakeData, d => d.value)])
    .range([chartHeight - 10, 10]);
    
  // Create line
  const line = d3.line()
    .x(d => x(d.month))
    .y(d => y(d.value))
    .curve(d3.curveMonotoneX);
    
  // Add line path
  chartSvg.append("path")
    .datum(fakeData)
    .attr("fill", "none")
    .attr("stroke", colorScale(stateData.value))
    .attr("stroke-width", 2)
    .attr("d", line);
    
  // Add dots
  chartSvg.selectAll("circle")
    .data(fakeData)
    .join("circle")
    .attr("cx", d => x(d.month))
    .attr("cy", d => y(d.value))
    .attr("r", 3)
    .attr("fill", colorScale(stateData.value));
}

// 6. Creating a choropleth for world map
function createWorldMap() {
  // Load world GeoJSON
  // d3.json("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json")
  //   .then(world => {
  //     // Create map with the loaded data
  //     renderWorldMap(world);
  //   });
  
  function renderWorldMap(world) {
    // Clear existing map
    mapGroup.selectAll("*").remove();
    
    // Create projection for world map
    const projection = d3.geoNaturalEarth1()
      .fitSize([width - margin.left - margin.right, height - margin.top - margin.bottom], 
               topojson.feature(world, world.objects.countries));
      
    const worldPath = d3.geoPath().projection(projection);
    
    // Create sample data
    const countryData = [
      {id: 840, name: "United States", value: 67},
      {id: 124, name: "Canada", value: 45},
      {id: 484, name: "Mexico", value: 32},
      // Add more countries
    ];
    
    // Create lookup for country data
    const countryDataById = {};
    countryData.forEach(d => {
      countryDataById[d.id] = d;
    });
    
    // Draw countries
    const countries = topojson.feature(world, world.objects.countries).features;
    
    mapGroup.selectAll("path.country")
      .data(countries)
      .join("path")
      .attr("class", "country")
      .attr("d", worldPath)
      .attr("fill", d => {
        const country = countryDataById[d.id];
        return country ? colorScale(country.value) : "#ccc";
      })
      .attr("stroke", "#fff")
      .attr("stroke-width", 0.5)
      .on("mouseover", function(event, d) {
        const country = countryDataById[d.id];
        if (!country) return;
        
        // Highlight country
        d3.select(this)
          .attr("stroke", "#333")
          .attr("stroke-width", 1.5);
          
        // Show tooltip
        tooltip
          .transition()
          .duration(200)
          .style("opacity", 0.9);
          
        tooltip
          .html(`<strong>${country.name}</strong><br>Value: ${country.value}`)
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", function() {
        d3.select(this)
          .attr("stroke", "#fff")
          .attr("stroke-width", 0.5);
          
        tooltip
          .transition()
          .duration(500)
          .style("opacity", 0);
      });
      
    // Add country labels (optional and can be overwhelming for world map)
    mapGroup.selectAll("text")
      .data(countries.filter(d => {
        const country = countryDataById[d.id];
        return country && country.value > 50; // Only label important countries
      }))
      .join("text")
      .attr("transform", d => `translate(${worldPath.centroid(d)})`)
      .attr("text-anchor", "middle")
      .attr("font-size", "8px")
      .text(d => {
        const country = countryDataById[d.id];
        return country ? country.name : "";
      });
  }
}
```### Force-Directed Graph
```javascript
// Sample network data
const nodes = [
  {id: "A", group: 1, size: 20},
  {id: "B", group: 1, size: 15},
  {id: "C", group: 2, size: 25},
  {id: "D", group: 2, size: 10},
  {id: "E", group: 3, size: 18},
  {id: "F", group: 3, size: 12},
  {id: "G", group: 4, size: 22}
];

const links = [
  {source: "A", target: "B", weight: 5},
  {source: "A", target: "C", weight: 2},
  {source: "B", target: "C", weight: 3},
  {source: "C", target: "D", weight: 1},
  {source: "C", target: "E", weight: 4},
  {source: "D", target: "E", weight: 7},
  {source: "E", target: "F", weight: 5},
  {source: "F", target: "G", weight: 3},
  {source: "G", target: "A", weight: 2}
];

// Set up dimensions
const width = 600;
const height = 500;

// Color scale for groups
const color = d3.scaleOrdinal(d3.schemeCategory10);

// Create SVG container
const svg = d3.select("#chart")
  .append("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("style", "max-width: 100%; height: auto;");

// Add arrowhead marker for directed links (optional)
svg.append("defs").append("marker")
  .attr("id", "arrowhead")
  .attr("viewBox", "0 -5 10 10")
  .attr("refX", 20)  // Position after the node radius
  .attr("refY", 0)
  .attr("markerWidth", 6)
  .attr("markerHeight", 6)
  .attr("orient", "auto")
  .append("path")
    .attr("d", "M0,-5L10,0L0,5")
    .attr("fill", "#999");

// Create a group for all graph elements
const g = svg.append("g");

// Add zoom functionality
const zoom = d3.zoom()
  .scaleExtent([0.1, 4])
  .on("zoom", (event) => {
    g.attr("transform", event.transform);
  });

svg.call(zoom);

// Create tooltip
const tooltip = d3.select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("background", "white")
  .style("border", "1px solid #ddd")
  .style("border-radius", "3px")
  .style("padding", "8px")
  .style("pointer-events", "none")
  .style("opacity", 0);

// Create the force simulation
const simulation = d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links).id(d => d.id).distance(d => 100 - d.weight * 8))
  .force("charge", d3.forceManyBody().strength(-300))
  .force("center", d3.forceCenter(width / 2, height / 2))
  .force("collide", d3.forceCollide().radius(d => d.size + 5).iterations(2));

// Add links
const link = g.append("g")
  .attr("class", "links")
  .selectAll("line")
  .data(links)
  .join("line")
    .attr("stroke", "#999")
    .attr("stroke-opacity", 0.6)
    .attr("stroke-width", d => Math.sqrt(d.weight))
    //.attr("marker-end", "url(#arrowhead)");  // Uncomment for directed graph

// Add nodes
const node = g.append("g")
  .attr("class", "nodes")
  .selectAll("circle")
  .data(nodes)
  .join("circle")
    .attr("r", d => d.size / 2)
    .attr("fill", d => color(d.group))
    .attr("stroke", "#fff")
    .attr("stroke-width", 1.5)
    .call(drag(simulation));

// Add labels
const label = g.append("g")
  .attr("class", "labels")
  .selectAll("text")
  .data(nodes)
  .join("text")
    .attr("text-anchor", "middle")
    .attr("dy", ".35em")
    .attr("font-size", "10px")
    .attr("font-weight", "bold")
    .text(d => d.id);

// Define drag behavior
function drag(simulation) {
  function dragstarted(event, d) {
    if (!event.active) simulation.alphaTarget(0.3).restart();
    d.fx = d.x;
    d.fy = d.y;
  }
  
  function dragged(event, d) {
    d.fx = event.x;
    d.fy = event.y;
  }
  
  function dragended(event, d) {
    if (!event.active) simulation.alphaTarget(0);
    // Uncomment to allow nodes to move again after drag
    // d.fx = null;
    // d.fy = null;
  }
  
  return d3.drag()
    .on("start", dragstarted)
    .on("drag", dragged)
    .on("end", dragended);
}

// Add interactive events
node
  .on("mouseover", function(event, d) {
    // Highlight connected links
    link
      .style("stroke", l => (l.source.id === d.id || l.target.id === d.id) ? "#ff3300" : "#999")
      .style("stroke-opacity", l => (l.source.id === d.id || l.target.id === d.id) ? 1 : 0.3);
      
    // Highlight connected nodes
    node
      .style("opacity", n => isConnected(d, n) ? 1 : 0.3)
      .style("stroke", n => isConnected(d, n) ? "#ff3300" : "#fff")
      .style("stroke-width", n => isConnected(d, n) ? 3 : 1.5);
      
    // Highlight this node
    d3.select(this)
      .style("stroke", "#ff3300")
      .style("stroke-width", 3);
      
    // Show tooltip
    tooltip
      .transition()
      .duration(200)
      .style("opacity", 0.9);
      
    tooltip
      .html(`Node: ${d.id}<br>Group: ${d.group}<br>Size: ${d.size}`)
      .style("left", (event.pageX + 10) + "px")
      .style("top", (event.pageY - 28) + "px");
  })
  .on("mouseout", function() {
    // Reset link styles
    link
      .style("stroke", "#999")
      .style("stroke-opacity", 0.6);
      
    // Reset node styles
    node
      .style("opacity", 1)
      .style("stroke", "#fff")
      .style("stroke-width", 1.5);
      
    // Hide tooltip
    tooltip
      .transition()
      .duration(500)
      .style("opacity", 0);
  });

// Helper function to check node connections
function isConnected(a, b) {
  if (a.id === b.id) return true;  // Same node
  
  // Check if there's a link between them
  return links.some(l => 
    (l.source.id === a.id && l.target.id === b.id) || 
    (l.source.id === b.id && l.target.id === a.id)
  );
}

// Run the simulation
simulation.on("tick", () => {
  // Update link positions
  link
    .attr("x1", d => d.source.x)
    .attr("y1", d => d.source.y)
    .attr("x2", d => d.target.x)
    .attr("y2", d => d.target.y);
    
  // Update node positions
  node
    .attr("cx", d => d.x)
    .attr("cy", d => d.y);
    
  // Update label positions
  label
    .attr("x", d => d.x)
    .attr("y", d => d.y);
});

// Add legend
const legend = svg.append("g")
  .attr("class", "legend")
  .attr("transform", "translate(20, 20)");

// Add legend items
const legendItems = legend.selectAll(".legend-item")
  .data([1, 2, 3, 4])  // Group IDs
  .join("g")
    .attr("class", "legend-item")
    .attr("transform", (d, i) => `translate(0, ${i * 20})`);

legendItems.append("circle")
  .attr("r", 6)
  .attr("fill", d => color(d));

legendItems.append("text")
  .attr("x", 12)
  .attr("y", 4)
  .text(d => `Group ${d}`);

// Add restart button for simulation
d3.select("#chart")
  .append("button")
  .text("Restart Simulation")
  .style("margin-top", "10px")
  .on("click", () => {
    // Reset node positions
    nodes.forEach(d => {
      d.fx = null;
      d.fy = null;
    });
    
    // Restart with high alpha
    simulation.alpha(1).restart();
  });

// Enhancements and variations

// 1. Bounded forces to keep nodes within view
function boundedForce() {
  const padding = 20;
  
  function force(alpha) {
    nodes.forEach(d => {
      // Calculate distance from edges
      const distLeft = d.x - padding;
      const distRight = width - padding - d.x;
      const distTop = d.y - padding;
      const distBottom = height - padding - d.y;
      
      // Apply forces if too close to edge
      if (distLeft < 0) d.vx += Math.abs(distLeft) * 0.1 * alpha;
      if (distRight < 0) d.vx -= Math.abs(distRight) * 0.1 * alpha;
      if (distTop < 0) d.vy += Math.abs(distTop) * 0.1 * alpha;
      if (distBottom < 0) d.vy -= Math.abs(distBottom) * 0.1 * alpha;
    });
  }
  
  return force;
}

simulation.force("bounds", boundedForce());

// 2. Display edge weights
function showEdgeWeights() {
  g.append("g")
    .attr("class", "edge-labels")
    .selectAll("text")
    .data(links)
    .join("text")
      .attr("font-size", "8px")
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("fill", "#666")
      .text(d => d.weight)
      .attr("x", d => (d.source.x + d.target.x) / 2)
      .attr("y", d => (d.source.y + d.target.y) / 2);
      
  // Update in tick function
  simulation.on("tick", () => {
    // ... existing tick code ...
    
    // Update edge weight labels
    svg.selectAll(".edge-labels text")
      .attr("x", d => (d.source.x + d.target.x) / 2)
      .attr("y", d => (d.source.y + d.target.y) / 2);
  });
}

// 3. Curved links for bidirectional relationships
function createCurvedLinks() {
  // Remove straight links
  link.remove();
  
  // Create curved links
  link = g.append("g")
    .attr("class", "links")
    .selectAll("path")
    .data(links)
    .join("path")
      .attr("fill", "none")
      .attr("stroke", "#999")
      .attr("stroke-opacity", 0.6)
      .attr("stroke-width", d => Math.sqrt(d.weight));
      
  // Update in tick function
  simulation.on("tick", () => {
    // ... existing tick code ...
    
    // Update curved paths
    link.attr("d", d => {
      const dx = d.target.x - d.source.x;
      const dy = d.target.y - d.source.y;
      const dr = Math.sqrt(dx * dx + dy * dy) * 2;  // Curve radius
      
      // Check if bidirectional
      const isBidirectional = links.some(l => 
        (l.source.id === d.target.id && l.target.id === d.source.id)
      );
      
      if (isBidirectional) {
        return `M${d.source.x},${d.source.y}A${dr},${dr} 0 0,1 ${d.target.x},${d.target.y}`;
      } else {
        return `M${d.source.x},${d.source.y}L${d.target.x},${d.target.y}`;
      }
    });
  });
}

// 4. Community detection visualization
function detectCommunities() {
  // Simple implementation of Louvain algorithm (not fully optimized)
  function findCommunities(nodes, links) {
    // Initialize each node to its own community
    nodes.forEach((node, i) => {
      node.community = i;
    });
    
    // Helper function to calculate modularity gain
    function modularityGain(node, community) {
      // Simplified implementation
      let gain = 0;
      links.forEach(link => {
        if (link.source.id === node.id && link.target.community === community) {
          gain += link.weight;
        } else if (link.target.id === node.id && link.source.community === community) {
          gain += link.weight;
        }
      });
      return gain;
    }
    
    // Run 3 iterations (simplified)
    for (let iter = 0; iter < 3; iter++) {
      let changes = false;
      
      nodes.forEach(node => {
        // Find connected communities
        const connectedCommunities = new Set();
        links.forEach(link => {
          if (link.source.id === node.id) {
            connectedCommunities.add(link.target.community);
          } else if (link.target.id === node.id) {
            connectedCommunities.add(link.source.community);
          }
        });
        
        // Find best community
        let bestCommunity = node.community;
        let bestGain = 0;
        
        connectedCommunities.forEach(community => {
          const gain = modularityGain(node, community);
          if (gain > bestGain) {
            bestGain = gain;
            bestCommunity = community;
          }
        });
        
        // Update community if better found
        if (bestCommunity !== node.community) {
          node.community = bestCommunity;
          changes = true;
        }
      });
      
      if (!changes) break;
    }
    
    // Map to sequential community IDs
    const communityMap = {};
    let nextCommunity = 0;
    
    nodes.forEach(node => {
      if (communityMap[node.community] === undefined) {
        communityMap[node.community] = nextCommunity++;
      }
      node.community = communityMap[node.community];
    });
    
    return nodes;
  }
  
  // Detect communities
  findCommunities(nodes, links);
  
  // Update node colors based on communities
  node.attr("fill", d => color(d.community));
  
  // Update legend
  const communityCount = d3.max(nodes, d => d.community) + 1;
  
  legend.selectAll("*").remove();
  
  const communityLegend = legend.selectAll(".legend-item")
    .data(d3.range(communityCount))
    .join("g")
      .attr("class", "legend-item")
      .attr("transform", (d, i) => `translate(0, ${i * 20})`);
      
  communityLegend.append("circle")
    .attr("r", 6)
    .attr("fill", d => color(d));
    
  communityLegend.append("text")
    .attr("x", 12)
    .attr("y", 4)
    .text(d => `Community ${d+1}`);
}

// 5. Node clustering by group
function clusterByGroup() {
  // Remove center force and add group forces
  simulation.force("center", null);
  
  // Get unique groups
  const groups = [...new Set(nodes.map(d => d.group))];
  
  // Create positions for each group
  const groupPositions = {};
  const angleStep = 2 * Math.PI / groups.length;
  const radius = Math.min(width, height) / 3;
  
  groups.forEach((group, i) => {
    const angle = i * angleStep;
    groupPositions[group] = {
      x: width / 2 + radius * Math.cos(angle),
      y: height / 2 + radius * Math.sin(angle)
    };
  });
  
  // Add force to pull nodes towards their group's position
  function groupForce(alpha) {
    nodes.forEach(d => {
      const target = groupPositions[d.group];
      d.vx += (target.x - d.x) * 0.1 * alpha;
      d.vy += (target.y - d.y) * 0.1 * alpha;
    });
  }
  
  simulation.force("group", groupForce);
  simulation.alpha(1).restart();
  
  // Add group labels
  g.append("g")
    .attr("class", "group-labels")
    .selectAll("text")
    .data(groups)
    .join("text")
      .attr("x", d => groupPositions[d].x)
      .attr("y", d => groupPositions[d].y - radius / 2 - 10)
      .attr("text-anchor", "middle")
      .attr("font-size", "14px")
      .attr("font-weight", "bold")
      .text(d => `Group ${d}`);
}
```### Hierarchical Tree
```javascript
// Sample hierarchical data
const data = {
  name: "Root",
  children: [
    {
      name: "Branch A",
      children: [
        { name: "Leaf A1", value: 10 },
        { name: "Leaf A2", value: 15 }
      ]
    },
    {
      name: "Branch B",
      children: [
        { name: "Leaf B1", value: 20 },
        { name: "Leaf B2", value: 25 },
        { 
          name: "Sub-branch B3",
          children: [
            { name: "Leaf B3a", value: 12 },
            { name: "Leaf B3b", value: 18 }
          ]
        }
      ]
    }
  ]
};

// Set up dimensions
const margin = {top: 40, right: 90, bottom: 50, left: 90};
const width = 800 - margin.left - margin.right;
const height = 500 - margin.top - margin.bottom;

// Create SVG container
const svg = d3.select("#chart")
  .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

// Create hierarchy
const root = d3.hierarchy(data);

// Assign positions
const treeLayout = d3.tree()
  .size([width, height]);

// Generate tree layout
treeLayout(root);

// Define line generator for links
const linkGenerator = d3.linkVertical()
  .x(d => d.x)
  .y(d => d.y);

// Create links
svg.selectAll(".link")
  .data(root.links())
  .join("path")
    .attr("class", "link")
    .attr("d", linkGenerator)
    .attr("fill", "none")
    .attr("stroke", "#aaa")
    .attr("stroke-width", 1.5);

// Create nodes
const nodes = svg.selectAll(".node")
  .data(root.descendants())
  .join("g")
    .attr("class", d => `node ${d.children ? "node-branch" : "node-leaf"}`)
    .attr("transform", d => `translate(${d.x},${d.y})`);

// Add circles to nodes
nodes.append("circle")
  .attr("r", d => d.children ? 8 : 6)
  .attr("fill", d => d.children ? "#69b3a2" : "#3498db")
  .attr("stroke", "#fff")
  .attr("stroke-width", 1.5);

// Add text labels
nodes.append("text")
  .attr("dy", d => d.children ? -15 : 15)
  .attr("x", d => d.children && d.data.name.length > 10 ? -10 : 0)
  .attr("text-anchor", d => d.children && d.data.name.length > 10 ? "end" : "middle")
  .text(d => d.data.name)
  .attr("font-size", "12px")
  .attr("font-weight", d => d.children ? "bold" : "normal");

// Add tooltip for more information
const tooltip = d3.select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("background", "white")
  .style("border", "1px solid #ddd")
  .style("border-radius", "3px")
  .style("padding", "8px")
  .style("pointer-events", "none")
  .style("opacity", 0);

// Add interactivity
nodes
  .on("mouseover", function(event, d) {
    // Highlight node
    d3.select(this).select("circle")
      .attr("r", d => d.children ? 10 : 8)
      .attr("stroke", "#333");
      
    // Show tooltip
    tooltip
      .transition()
      .duration(200)
      .style("opacity", 0.9);
      
    tooltip
      .html(`Name: ${d.data.name}<br>${d.data.value ? `Value: ${d.data.value}` : ""}`)
      .style("left", (event.pageX + 10) + "px")
      .style("top", (event.pageY - 28) + "px");
  })
  .on("mouseout", function(event, d) {
    // Reset node
    d3.select(this).select("circle")
      .attr("r", d => d.children ? 8 : 6)
      .attr("stroke", "#fff");
      
    // Hide tooltip
    tooltip
      .transition()
      .duration(500)
      .style("opacity", 0);
  });

// Add animation on creation
function animateTree() {
  // Animate links
  svg.selectAll(".link")
    .attr("stroke-dasharray", function() {
      const length = this.getTotalLength();
      return `${length} ${length}`;
    })
    .attr("stroke-dashoffset", function() {
      return this.getTotalLength();
    })
    .transition()
    .duration(1000)
    .attr("stroke-dashoffset", 0);
    
  // Animate nodes
  nodes.select("circle")
    .attr("r", 0)
    .transition()
    .duration(500)
    .delay((d, i) => i * 50 + 500)
    .attr("r", d => d.children ? 8 : 6);
    
  // Animate text
  nodes.select("text")
    .style("opacity", 0)
    .transition()
    .duration(500)
    .delay((d, i) => i * 50 + 750)
    .style("opacity", 1);
}

// Call animation
animateTree();

// Radial tree variation
function createRadialTree() {
  // Clear existing elements
  svg.selectAll(".link, .node").remove();
  
  // Create new layout for radial tree
  const radialLayout = d3.tree()
    .size([2 * Math.PI, Math.min(width, height) / 2 - 80]);  // Angle and radius
    
  // Apply layout
  const radialRoot = d3.hierarchy(data);
  radialLayout(radialRoot);
  
  // Position group at center
  const g = svg.append("g")
    .attr("transform", `translate(${width / 2},${height / 2})`);
    
  // Create links with radial coordinates
  const radialLink = d3.linkRadial()
    .angle(d => d.x)
    .radius(d => d.y);
    
  g.selectAll(".link")
    .data(radialRoot.links())
    .join("path")
      .attr("class", "link")
      .attr("d", radialLink)
      .attr("fill", "none")
      .attr("stroke", "#aaa")
      .attr("stroke-width", 1.5);
      
  // Create nodes with radial coordinates
  const node = g.selectAll(".node")
    .data(radialRoot.descendants())
    .join("g")
      .attr("class", d => `node ${d.children ? "node-branch" : "node-leaf"}`)
      .attr("transform", d => `translate(${radialPoint(d.x, d.y)})`);
      
  // Helper function for radial coordinates
  function radialPoint(x, y) {
    return [(y = +y) * Math.cos(x -= Math.PI / 2), y * Math.sin(x)];
  }
      
  // Add circles to nodes
  node.append("circle")
    .attr("r", d => d.children ? 8 : 6)
    .attr("fill", d => d.children ? "#69b3a2" : "#3498db")
    .attr("stroke", "#fff")
    .attr("stroke-width", 1.5);
      
  // Add text labels with proper rotation
  node.append("text")
    .attr("dy", "0.31em")
    .attr("x", d => d.x < Math.PI === !d.children ? 10 : -10)
    .attr("text-anchor", d => d.x < Math.PI === !d.children ? "start" : "end")
    .attr("transform", d => {
      const angle = d.x * 180 / Math.PI - 90;
      const flip = angle > 90 ? 180 : 0;
      return `rotate(${angle}) translate(${d.children ? -15 : 15}) rotate(${flip})`;
    })
    .text(d => d.data.name)
    .attr("font-size", "10px");
}

// Collapsible tree implementation
function createCollapsibleTree() {
  // Reset SVG
  svg.selectAll("*").remove();
  
  // Create hierarchy with all nodes expanded initially
  const root = d3.hierarchy(data);
  
  // Set initial positions and toggle children function
  root.x0 = width / 2;
  root.y0 = 0;
  
  // Collapse function to hide children
  function collapse(d) {
    if (d.children) {
      d._children = d.children;
      d._children.forEach(collapse);
      d.children = null;
    }
  }
  
  // Expand a few nodes initially
  root.children.forEach(collapse);
  
  // Keep track of toggled nodes
  const toggled = new Set();
  toggled.add(root.data.name);  // Root is expanded
  
  // Update tree display
  update(root);
  
  // Update function for interactive tree
  function update(source) {
    // Create tree layout
    const treeLayout = d3.tree().size([width, height]);
    
    // Compute new tree layout
    const nodes = treeLayout(root);
    
    // Get all node descendants
    const allNodes = root.descendants();
    const allLinks = root.links();
    
    // Set fixed depth Y position
    allNodes.forEach(d => {
      d.y = d.depth * 100;  // Fixed distance between tree levels
    });
    
    // Update nodes
    const node = svg.selectAll(".node")
      .data(allNodes, d => d.id || (d.id = `node-${d.data.name}-${Math.random()}`));
      
    // Enter new nodes at parent's previous position
    const nodeEnter = node.enter().append("g")
      .attr("class", "node")
      .attr("transform", d => `translate(${source.x0},${source.y0})`)
      .on("click", clicked);
      
    // Add circle to nodes
    nodeEnter.append("circle")
      .attr("r", 6)
      .attr("fill", d => d._children ? "#E8B39D" : d.children ? "#69b3a2" : "#3498db")
      .attr("stroke", "#fff")
      .attr("stroke-width", 1.5);
      
    // Add node text
    nodeEnter.append("text")
      .attr("dy", d => d.children || d._children ? -15 : 15)
      .attr("x", 0)
      .attr("text-anchor", "middle")
      .text(d => d.data.name)
      .attr("font-size", "12px")
      .style("fill-opacity", 0);
      
    // Update nodes with transition to new position
    const nodeUpdate = nodeEnter.merge(node)
      .transition()
      .duration(750)
      .attr("transform", d => `translate(${d.x},${d.y})`);
      
    // Update node appearance based on state
    nodeUpdate.select("circle")
      .attr("r", 6)
      .attr("fill", d => d._children ? "#E8B39D" : d.children ? "#69b3a2" : "#3498db");
      
    nodeUpdate.select("text")
      .style("fill-opacity", 1);
      
    // Exit nodes with transition to parent's new position
    const nodeExit = node.exit()
      .transition()
      .duration(750)
      .attr("transform", d => `translate(${source.x},${source.y})`)
      .remove();
      
    nodeExit.select("circle")
      .attr("r", 0);
      
    nodeExit.select("text")
      .style("fill-opacity", 0);
      
    // Update links
    const link = svg.selectAll(".link")
      .data(allLinks, d => d.target.id);
      
    // Enter new links at parent's previous position
    const linkEnter = link.enter().append("path")
      .attr("class", "link")
      .attr("d", d => {
        const o = {x: source.x0, y: source.y0};
        return diagonal({source: o, target: o});
      })
      .attr("fill", "none")
      .attr("stroke", "#aaa")
      .attr("stroke-width", 1.5);
      
    // Update links with transition
    linkEnter.merge(link)
      .transition()
      .duration(750)
      .attr("d", diagonal);
      
    // Exit links with transition to parent's new position
    link.exit()
      .transition()
      .duration(750)
      .attr("d", d => {
        const o = {x: source.x, y: source.y};
        return diagonal({source: o, target: o});
      })
      .remove();
      
    // Store old positions for transition
    allNodes.forEach(d => {
      d.x0 = d.x;
      d.y0 = d.y;
    });
    
    // Toggle children on click
    function clicked(event, d) {
      if (d.children) {
        d._children = d.children;
        d.children = null;
        toggled.delete(d.data.name);
      } else if (d._children) {
        d.children = d._children;
        d._children = null;
        toggled.add(d.data.name);
      }
      update(d);
    }
    
    // Custom diagonal connector for links
    function diagonal(link) {
      const source = link.source;
      const target = link.target;
      
      return d3.linkVertical()
        .x(d => d.x)
        .y(d => d.y)
        ({source, target});
    }
  }
}
```### Heatmap
```javascript
// Sample data
const data = [
  {row: 0, col: 0, value: 10},
  {row: 0, col: 1, value: 25},
  {row: 0, col: 2, value: 30},
  {row: 1, col: 0, value: 5},
  {row: 1, col: 1, value: 15},
  {row: 1, col: 2, value: 40},
  {row: 2, col: 0, value: 35},
  {row: 2, col: 1, value: 20},
  {row: 2, col: 2, value: 12}
];

// Row and column labels
const rowLabels = ["Row A", "Row B", "Row C"];
const colLabels = ["Col X", "Col Y", "Col Z"];

// Set up dimensions
const margin = {top: 50, right: 50, bottom: 50, left: 80};
const width = 500 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

// Cell dimensions
const cellWidth = width / colLabels.length;
const cellHeight = height / rowLabels.length;

// Create SVG container
const svg = d3.select("#chart")
  .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

// Create scales
const x = d3.scaleBand()
  .domain(d3.range(colLabels.length))
  .range([0, width])
  .padding(0.05);

const y = d3.scaleBand()
  .domain(d3.range(rowLabels.length))
  .range([0, height])
  .padding(0.05);

// Color scale
const colorScale = d3.scaleSequential()
  .domain([0, d3.max(data, d => d.value)])
  .interpolator(d3.interpolateViridis);

// Add heatmap cells
svg.selectAll("rect")
  .data(data)
  .join("rect")
    .attr("x", d => x(d.col))
    .attr("y", d => y(d.row))
    .attr("width", cellWidth)
    .attr("height", cellHeight)
    .attr("fill", d => colorScale(d.value))
    .attr("stroke", "#fff")
    .attr("stroke-width", 1);

// Add text labels to cells
svg.selectAll(".cellText")
  .data(data)
  .join("text")
    .attr("class", "cellText")
    .attr("x", d => x(d.col) + cellWidth / 2)
    .attr("y", d => y(d.row) + cellHeight / 2)
    .attr("text-anchor", "middle")
    .attr("dominant-baseline", "middle")
    .attr("font-size", "12px")
    .attr("fill", d => d.value > 25 ? "#fff" : "#000") // Contrast with background
    .text(d => d.value);

// Add x-axis (column) labels
svg.selectAll(".colLabel")
  .data(colLabels)
  .join("text")
    .attr("class", "colLabel")
    .attr("x", (d, i) => x(i) + cellWidth / 2)
    .attr("y", -10)
    .attr("text-anchor", "middle")
    .attr("font-size", "12px")
    .attr("font-weight", "bold")
    .text(d => d);

// Add y-axis (row) labels
svg.selectAll(".rowLabel")
  .data(rowLabels)
  .join("text")
    .attr("class", "rowLabel")
    .attr("x", -10)
    .attr("y", (d, i) => y(i) + cellHeight / 2)
    .attr("text-anchor", "end")
    .attr("dominant-baseline", "middle")
    .attr("font-size", "12px")
    .attr("font-weight", "bold")
    .text(d => d);

// Add title
svg.append("text")
  .attr("x", width / 2)
  .attr("y", -30)
  .attr("text-anchor", "middle")
  .attr("font-size", "16px")
  .attr("font-weight", "bold")
  .text("Heatmap Example");

// Add color legend
const legendWidth = 200;
const legendHeight = 20;

const legendX = d3.scaleLinear()
  .domain([0, d3.max(data, d => d.value)])
  .range([0, legendWidth]);

const legendXAxis = d3.axisBottom(legendX)
  .ticks(5)
  .tickSize(6);

// Legend container
const legend = svg.append("g")
  .attr("class", "legend")
  .attr("transform", `translate(${width - legendWidth - 10},${height + 30})`);

// Gradient for legend
const defs = svg.append("defs");
const linearGradient = defs.append("linearGradient")
  .attr("id", "heatmap-gradient")
  .attr("x1", "0%")
  .attr("x2", "100%")
  .attr("y1", "0%")
  .attr("y2", "0%");

// Define gradient stops
const gradientStops = d3.range(0, 1.1, 0.1);
gradientStops.forEach(stop => {
  linearGradient.append("stop")
    .attr("offset", stop)
    .attr("stop-color", colorScale(stop * d3.max(data, d => d.value)))
    .attr("stop-opacity", 1);
});

// Add legend rectangle
legend.append("rect")
  .attr("width", legendWidth)
  .attr("height", legendHeight)
  .style("fill", "url(#heatmap-gradient)");

// Add legend axis
legend.append("g")
  .attr("transform", `translate(0,${legendHeight})`)
  .call(legendXAxis);

// Add tooltip
const tooltip = d3.select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("background", "white")
  .style("border", "1px solid #ddd")
  .style("border-radius", "3px")
  .style("padding", "8px")
  .style("pointer-events", "none")
  .style("opacity", 0);

// Add interactivity
svg.selectAll("rect")
  .on("mouseover", function(event, d) {
    // Highlight cell
    d3.select(this)
      .attr("stroke", "#333")
      .attr("stroke-width", 2);
      
    // Show tooltip
    tooltip
      .transition()
      .duration(200)
      .style("opacity", 0.9);
      
    tooltip
      .html(`Row: ${rowLabels[d.row]}<br>Column: ${colLabels[d.col]}<br>Value: ${d.value}`)
      .style("left", (event.pageX + 10) + "px")
      .style("top", (event.pageY - 28) + "px");
  })
  .on("mouseout", function() {
    // Reset cell
    d3.select(this)
      .attr("stroke", "#fff")
      .attr("stroke-width", 1);
      
    // Hide tooltip
    tooltip
      .transition()
      .duration(500)
      .style("opacity", 0);
  });

// Animate heatmap on creation
function animateHeatmap() {
  svg.selectAll("rect")
    .attr("opacity", 0)
    .transition()
    .duration(500)
    .delay((d, i) => i * 50)
    .attr("opacity", 1);
    
  svg.selectAll(".cellText")
    .attr("opacity", 0)
    .transition()
    .duration(500)
    .delay((d, i) => i * 50 + 250)
    .attr("opacity", 1);
}

// Call animation
animateHeatmap();

// Update heatmap with new data
function updateHeatmap(newData) {
  // Update color scale
  colorScale.domain([0, d3.max(newData, d => d.value)]);
  
  // Update cells
  svg.selectAll("rect")
    .data(newData)
    .transition()
    .duration(750)
    .attr("fill", d => colorScale(d.value));
    
  // Update text labels
  svg.selectAll(".cellText")
    .data(newData)
    .transition()
    .duration(750)
    .attr("fill", d => d.value > 25 ? "#fff" : "#000")
    .text(d => d.value);
}

// Matrix with custom data
function createCorrelationMatrix(data) {
  // Create symmetric matrix for correlation
  const matrix = [];
  const n = 5;  // Number of variables
  
  // Generate random correlation values
  for (let i = 0; i < n; i++) {
    matrix[i] = [];
    for (let j = 0; j < n; j++) {
      if (i === j) {
        matrix[i][j] = 1;  // Diagonal is always 1
      } else if (j < i) {
        matrix[i][j] = matrix[j][i];  // Mirror values across diagonal
      } else {
        matrix[i][j] = Math.round((Math.random() * 2 - 1) * 100) / 100;  // Random between -1 and 1
      }
    }
  }
  
  // Convert to flat format
  const flatData = [];
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      flatData.push({row: i, col: j, value: matrix[i][j]});
    }
  }
  
  // Update color scale for correlation (-1 to 1)
  const corrColorScale = d3.scaleSequential()
    .domain([-1, 1])
    .interpolator(d3.interpolateRdBu);
    
  // Update cells
  svg.selectAll("rect")
    .data(flatData)
    .join("rect")
      .attr("x", d => x(d.col))
      .attr("y", d => y(d.row))
      .attr("width", cellWidth)
      .attr("height", cellHeight)
      .attr("fill", d => corrColorScale(d.value))
      .attr("stroke", "#fff")
      .attr("stroke-width", 1);
      
  // Update text
  svg.selectAll(".cellText")
    .data(flatData)
    .join("text")
      .attr("class", "cellText")
      .attr("x", d => x(d.col) + cellWidth / 2)
      .attr("y", d => y(d.row) + cellHeight / 2)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("font-size", "12px")
      .attr("fill", d => Math.abs(d.value) > 0.5 ? "#fff" : "#000")
      .text(d => d.value.toFixed(2));
}
```### Pie Chart
```javascript
// Sample data
const data = [
  { name: "Category A", value: 30 },
  { name: "Category B", value: 15 },
  { name: "Category C", value: 45 },
  { name: "Category D", value: 10 }
];

// Set up dimensions
const width = 500;
const height = 400;
const radius = Math.min(width, height) / 2;

// Create SVG container
const svg = d3.select("#chart")
  .append("svg")
    .attr("width", width)
    .attr("height", height)
  .append("g")
    .attr("transform", `translate(${width / 2}, ${height / 2})`);

// Color scale
const color = d3.scaleOrdinal()
  .domain(data.map(d => d.name))
  .range(d3.schemeCategory10);

// Pie layout generator
const pie = d3.pie()
  .value(d => d.value)
  .sort(null);                             // Don't sort by value

// Arc generator
const arc = d3.arc()
  .innerRadius(0)                          // 0 for pie chart
  .outerRadius(radius - 10);               // Outer radius with padding

// Label arc (for text positioning)
const labelArc = d3.arc()
  .innerRadius(radius * 0.6)               // Position labels at 60% radius
  .outerRadius(radius * 0.6);              // Match inner radius

// Create pie slices
const arcs = svg.selectAll(".arc")
  .data(pie(data))
  .join("g")
    .attr("class", "arc");

// Add path for each slice
arcs.append("path")
  .attr("d", arc)
  .attr("fill", d => color(d.data.name))
  .attr("stroke", "white")
  .attr("stroke-width", 1)
  .style("transition", "opacity 0.3s");

// Add text labels
arcs.append("text")
  .attr("transform", d => `translate(${labelArc.centroid(d)})`)
  .attr("text-anchor", "middle")
  .attr("dy", "0.35em")
  .text(d => d.data.name)
  .style("font-size", "12px")
  .style("font-weight", "bold")
  .style("fill", "#333");

// Add percentage labels
arcs.append("text")
  .attr("transform", d => {
    const [x, y] = arc.centroid(d);
    const offset = radius * 0.35;
    const angle = (d.startAngle + d.endAngle) / 2;
    const position = [
      x * offset / radius,
      y * offset / radius
    ];
    return `translate(${position})`;
  })
  .attr("text-anchor", "middle")
  .attr("dy", "0.35em")
  .text(d => `${Math.round((d.endAngle - d.startAngle) / (2 * Math.PI) * 100)}%`)
  .style("font-size", "10px")
  .style("fill", "white");
  
// Add total in center
svg.append("text")
  .attr("text-anchor", "middle")
  .attr("dy", "0.35em")
  .text(() => {
    const total = d3.sum(data, d => d.value);
    return `Total: ${total}`;
  })
  .style("font-size", "16px")
  .style("font-weight", "bold");

// Interactive highlights
arcs
  .on("mouseover", function(event, d) {
    // Highlight current slice
    d3.select(this).select("path")
      .transition()
      .duration(200)
      .attr("opacity", 1)
      .attr("d", d3.arc()
        .innerRadius(0)
        .outerRadius(radius + 10)          // Expand on hover
      );
      
    // Fade other slices
    arcs.filter(p => p !== d)
      .select("path")
      .transition()
      .duration(200)
      .attr("opacity", 0.5);
  })
  .on("mouseout", function() {
    // Reset all slices
    arcs.select("path")
      .transition()
      .duration(200)
      .attr("opacity", 1)
      .attr("d", arc);
  });

// Add legend
const legend = svg.append("g")
  .attr("class", "legend")
  .attr("transform", `translate(${radius + 30}, ${-radius + 20})`);

const legendItems = legend.selectAll(".legend-item")
  .data(data)
  .join("g")
    .attr("class", "legend-item")
    .attr("transform", (d, i) => `translate(0, ${i * 20})`);

legendItems.append("rect")
  .attr("width", 12)
  .attr("height", 12)
  .attr("fill", d => color(d.name));

legendItems.append("text")
  .attr("x", 20)
  .attr("y", 6)
  .attr("dy", "0.35em")
  .text(d => `${d.name} (${d.value})`)
  .style("font-size", "12px");

// Donut chart variation
function createDonutChart() {
  // Modify arc generator for donut
  const donutArc = d3.arc()
    .innerRadius(radius * 0.5)             // Inner radius creates hole
    .outerRadius(radius - 10);
    
  // Update paths
  arcs.select("path")
    .transition()
    .duration(750)
    .attr("d", donutArc);
    
  // Update label positions
  const donutLabelArc = d3.arc()
    .innerRadius(radius * 0.8)
    .outerRadius(radius * 0.8);
    
  arcs.select("text")
    .transition()
    .duration(750)
    .attr("transform", d => `translate(${donutLabelArc.centroid(d)})`);
}

// Animated pie chart
function animatePieChart() {
  // Create initial state with all values at 0
  const arcTween = function(d) {
    const interpolate = d3.interpolate({ startAngle: 0, endAngle: 0 }, d);
    return function(t) {
      return arc(interpolate(t));
    };
  };
  
  // Animate path
  arcs.select("path")
    .transition()
    .duration(1000)
    .attrTween("d", arcTween);
    
  // Fade in text
  arcs.selectAll("text")
    .style("opacity", 0)
    .transition()
    .delay(750)
    .duration(500)
    .style("opacity", 1);
}

// Update pie chart with new data
function updatePieChart(newData) {
  // Update arc data
  const arcUpdate = svg.selectAll(".arc")
    .data(pie(newData));
    
  // Exit
  arcUpdate.exit()
    .transition()
    .duration(500)
    .style("opacity", 0)
    .remove();
    
  // Enter
  const arcEnter = arcUpdate.enter()
    .append("g")
    .attr("class", "arc");
    
  arcEnter.append("path")
    .attr("d", arc)
    .attr("fill", d => color(d.data.name))
    .attr("stroke", "white")
    .attr("stroke-width", 1);
    
  arcEnter.append("text")
    .attr("transform", d => `translate(${labelArc.centroid(d)})`)
    .attr("text-anchor", "middle")
    .attr("dy", "0.35em")
    .text(d => d.data.name)
    .style("font-size", "12px");
    
  // Update existing arcs
  arcUpdate.select("path")
    .transition()
    .duration(750)
    .attrTween("d", function(d) {
      const current = this._current || { startAngle: 0, endAngle: 0 };
      const interpolate = d3.interpolate(current, d);
      this._current = interpolate(0);
      return function(t) {
        return arc(interpolate(t));
      };
    })
    .attr("fill", d => color(d.data.name));
    
  arcUpdate.select("text")
    .transition()
    .duration(750)
    .attr("transform", d => `translate(${labelArc.centroid(d)})`)
    .text(d => d.data.name);
}

// Exploded pie chart
function explodePie() {
  arcs.selectAll("path")
    .transition()
    .duration(750)
    .attrTween("d", function(d) {
      const current = this._current || d;
      const exploded = {
        ...d,
        startAngle: d.startAngle,
        endAngle: d.endAngle
      };
      
      // Calculate explosion offset
      const midAngle = (d.startAngle + d.endAngle) / 2;
      const offset = 20;
      exploded._exploded = true;
      exploded._x = Math.cos(midAngle) * offset;
      exploded._y = Math.sin(midAngle) * offset;
      
      const interpolate = d3.interpolate(current, exploded);
      this._current = interpolate(0);
      
      return function(t) {
        const values = interpolate(t);
        const customArc = d3.arc()
          .innerRadius(0)
          .outerRadius(radius - 10);
          
        // Apply translation
        if (values._exploded) {
          return d3.arc()
            .innerRadius(0)
            .outerRadius(radius - 10)
            .centroid(values)
            .map((p, i) => p + (i === 0 ? values._x : values._y))
            .join(",");
        } else {
          return customArc(values);
        }
      };
    });
}
```y-axis").call(d3.axisLeft(newY));
  
  // Update circles
  svg.selectAll("circle")
    .attr("cx", d => newX(d.x))
    .attr("cy", d => newY(d.y));
    
  // Update grid lines
  svg.select(".x-grid")
    .call(d3.axisBottom(newX)
      .tickSize(-height)
      .tickFormat(""));
      
  svg.select(".y-grid")
    .call(d3.axisLeft(newY)
      .tickSize(-width)
      .tickFormat(""));
}

// Bubble chart variation 
function createBubbleChart(data) {
  // Add title
  svg.append("text")
    .attr("x", width / 2)
    .attr("y", -margin.top / 2)
    .attr("text-anchor", "middle")
    .style("font-size", "16px")
    .style("font-weight", "bold")
    .text("Bubble Chart");
    
  // Rework data for packing
  const packData = {
    children: data.map(d => ({
      name: d.group,
      value: d.size
    }))
  };
  
  // Create pack layout
  const pack = d3.pack()
    .size([width, height])
    .padding(3);
    
  // Create hierarchy
  const root = d3.hierarchy(packData)
    .sum(d => d.value);
    
  // Generate layout
  const bubbles = pack(root).leaves();
  
  // Add bubbles
  svg.selectAll(".bubble")
    .data(bubbles)
    .join("circle")
      .attr("class", "bubble")
      .attr("cx", d => d.x)
      .attr("cy", d => d.y)
      .attr("r", d => d.r)
      .attr("fill", d => color(d.data.name))
      .attr("fill-opacity", 0.7)
      .attr("stroke", d => d3.rgb(color(d.data.name)).darker())
      .attr("stroke-width", 1);
      
  // Add labels
  svg.selectAll(".label")
    .data(bubbles)
    .join("text")
      .attr("class", "label")
      .attr("x", d => d.x)
      .attr("y", d => d.y)
      .attr("text-anchor", "middle")
      .attr("dy", "0.3em")
      .text(d => d.data.name)
      .style("font-size", d => Math.min(d.r * 0.6, 12))
      .style("fill", "#fff")
      .style("pointer-events", "none");
}

// Add brushing
function addBrushing() {
  // Add brush
  const brush = d3.brush()
    .extent([[0, 0], [width, height]])
    .on("start brush end", brushed);
    
  // Add brush to SVG
  svg.append("g")
    .attr("class", "brush")
    .call(brush);
    
  // Brushed function
  function brushed(event) {
    if (!event.selection) return;
    
    // Get selection bounds
    const [[x0, y0], [x1, y1]] = event.selection;
    
    // Update circle opacity based on selection
    svg.selectAll("circle")
      .attr("fill-opacity", d => {
        const cx = x(d.x);
        const cy = y(d.y);
        return (cx >= x0 && cx <= x1 && cy >= y0 && cy <= y1) ? 1 : 0.3;
      });
  }
}

// Different symbols for groups
function useSymbols() {
  // Map groups to symbol types
  const symbolMap = {
    "A": d3.symbolCircle,
    "B": d3.symbolTriangle,
    "C": d3.symbolSquare
  };
  
  // Create symbol generator
  const symbol = d3.symbol()
    .size(d => size(d.size) * 40)
    .type(d => symbolMap[d.group]);
    
  // Replace circles with symbols
  svg.selectAll("circle").remove();
  
  svg.selectAll("path")
    .data(data)
    .join("path")
      .attr("transform", d => `translate(${x(d.x)},${y(d.y)})`)
      .attr("d", symbol)
      .attr("fill", d => color(d.group))
      .attr("fill-opacity", 0.7)
      .attr("stroke", d => d3.rgb(color(d.group)).darker())
      .attr("stroke-width", 1);
}

// Animated transitions
function animateScatter(newData) {
  // Update scales
  x.domain([0, d3.max(newData, d => d.x) * 1.1]);
  y.domain([0, d3.max(newData, d => d.y) * 1.1]).nice();
  
  // Update axes with transition
  svg.select(".x-axis")
    .transition()
    .duration(750)
    .call(d3.axisBottom(x));
    
  svg.select(".y-axis")
    .transition()
    .duration(750)
    .call(d3.axisLeft(y));
    
  // Update circles with transition
  svg.selectAll("circle")
    .data(newData)
    .join(
      enter => enter.append("circle")
        .attr("cx", d => x(d.x))
        .attr("cy", height)
        .attr("r", 0)
        .attr("fill", d => color(d.group))
        .call(enter => enter.transition()
          .duration(750)
          .attr("cy", d => y(d.y))
          .attr("r", d => size(d.size))
        ),
      update => update
        .call(update => update.transition()
          .duration(750)
          .attr("cx", d => x(d.x))
          .attr("cy", d => y(d.y))
          .attr("r", d => size(d.size))
        ),
      exit => exit
        .call(exit => exit.transition()
          .duration(750)
          .attr("cy", height)
          .attr("r", 0)
          .remove()
        )
    );
}
```
  ### Interaction
```javascript
// Event handling
selection.on("mouseover", function(event, d) {
  d3.select(this).attr("fill", "red");     // 'this' refers to DOM element
});

// Common mouse events
selection.on("mouseover", handler);        // Mouse enters element
selection.on("mouseout", handler);         // Mouse leaves element
selection.on("mousemove", handler);        // Mouse moves within element
selection.on("mouseenter", handler);       // Mouse enters (no bubble)
selection.on("mouseleave", handler);       // Mouse leaves (no bubble)
selection.on("click", handler);            // Mouse click
selection.on("dblclick", handler);         // Double click

// Touch events
selection.on("touchstart", handler);       // Touch begins
selection.on("touchmove", handler);        // Touch moves
selection.on("touchend", handler);         // Touch ends

// Other events
selection.on("input", handler);            // Input value changes
selection.on("change", handler);           // Select/checkbox changes
selection.on("focus", handler);            // Element receives focus
selection.on("blur", handler);             // Element loses focus

// Event object properties
function handler(event, d) {
  event.type;                              // Event type (e.g., "click")
  event.target;                            // Target element
  event.currentTarget;                     // Current element (this)
  event.clientX;                           // Mouse X position (window)
  event.clientY;                           // Mouse Y position (window)
  event.pageX;                             // Mouse X position (document)
  event.pageY;                             // Mouse Y position (document)
  event.screenX;                           // Mouse X position (screen)
  event.screenY;                           // Mouse Y position (screen)
  event.offsetX;                           // Mouse X position (target element)
  event.offsetY;                           // Mouse Y position (target element)
  event.button;                            // Mouse button (0=left, 1=middle, 2=right)
  event.buttons;                           // Buttons pressed during event
  event.altKey;                            // Whether Alt key is pressed
  event.ctrlKey;                           // Whether Ctrl key is pressed
  event.metaKey;                           // Whether Meta key is pressed
  event.shiftKey;                          // Whether Shift key is pressed
  event.touches;                           // List of touches (touch events)
  event.changedTouches;                    // List of changed touches
  
  event.preventDefault();                  // Prevent default behavior
  event.stopPropagation();                 // Stop event propagation
}

// Get mouse position in SVG coordinates
function getMousePosition(event) {
  const svg = d3.select("svg").node();
  const pt = svg.createSVGPoint();
  pt.x = event.clientX;
  pt.y = event.clientY;
  return pt.matrixTransform(svg.getScreenCTM().inverse());
}

// Multiple event listeners
selection
  .on("mouseover", mouseoverHandler)
  .on("mouseout", mouseoutHandler)
  .on("click", clickHandler);

// Remove event listener
selection.on("click", null);               // Remove click handler
selection.on(".ns", null);                 // Remove all handlers with namespace
selection.on(".*", null);                  // Remove all handlers

// Event namespaces
selection.on("click.ns1", handler1);       // Add namespaced handler
selection.on("click.ns2", handler2);       // Add another namespaced handler
selection.on("click.ns1", null);           // Remove specific handler

// Dragging
const drag = d3.drag()
  .on("start", dragstarted)
  .on("drag", dragged)
  .on("end", dragended);

selection.call(drag);                      // Apply drag behavior

// Drag event handlers
function dragstarted(event, d) {
  d3.select(this).raise().classed("active", true);
  // Use event.x, event.y for position
}

function dragged(event, d) {
  d3.select(this)
    .attr("cx", d.x = event.x)
    .attr("cy", d.y = event.y);
}

function dragended(event, d) {
  d3.select(this).classed("active", false);
}

// Customize drag behavior
drag.filter(event => !event.button)        // Filter events (only left button)
  .subject(function(event, d) {            // Set subject being dragged
    return d;
  })
  .clickDistance(5)                        // Minimum distance for click
  .container(node);                        // Constrain to container

// Zooming
const zoom = d3.zoom()
  .scaleExtent([1, 10])                    // Min/max zoom scale
  .translateExtent([[0, 0], [width, height]]) // Constrain panning
  .on("zoom", zoomed);

svg.call(zoom);                            // Apply zoom behavior

function zoomed(event) {
  svg.attr("transform", event.transform);  // Apply zoom transform
}

// Zoom transform
const transform = d3.zoomIdentity
  .translate(x, y)                         // Translation
  .scale(k);                               // Scale

// Zoom methods
svg.call(zoom.transform, d3.zoomIdentity); // Reset zoom
svg.call(zoom.scaleBy, 1.2);               // Scale by factor
svg.call(zoom.scaleTo, 2);                 // Scale to specific scale
svg.call(zoom.translateBy, 10, 0);         // Translate by x,y

// Pan to specific element
function centerOn(element) {
  const bounds = element.node().getBBox();
  const dx = bounds.width;
  const dy = bounds.height;
  const x = bounds.x + dx/2;
  const y = bounds.y + dy/2;
  
  const scale = Math.min(1, Math.min(width/dx, height/dy));
  const transform = d3.zoomIdentity
    .translate(width/2, height/2)
    .scale(scale)
    .translate(-x, -y);
    
  svg.transition()
    .duration(750)
    .call(zoom.transform, transform);
}

// Brush (for selection)
const brush = d3.brush()
  .extent([[0, 0], [width, height]])       // Brush area
  .on("start brush end", brushed);

svg.append("g")
  .attr("class", "brush")
  .call(brush);

function brushed(event) {
  const selection = event.selection;
  if (!selection) return; // No selection
  
  const [[x0, y0], [x1, y1]] = selection;  // Selection coordinates
  
  // Filter elements in selection
  dots.classed("selected", d => {
    const x = xScale(d.x);
    const y = yScale(d.y);
    return x >= x0 && x <= x1 && y >= y0 && y <= y1;
  });
}

// Brush types
const xBrush = d3.brushX();                // 1D brush (x-axis)
const yBrush = d3.brushY();                // 1D brush (y-axis)
const xyBrush = d3.brush();                // 2D brush (x and y)

// Custom brushes
brush.handleSize(10)                       // Size of handles
  .keyModifiers(true)                      // Enable key modifiers
  .filter(event => !event.button)          // Filter events
  .touchable(true);                        // Enable touch events

// Tooltip (no built-in, common pattern)
const tooltip = d3.select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("opacity", 0)
  .style("position", "absolute")
  .style("pointer-events", "none");

// Show tooltip on hover
selection.on("mouseover", function(event, d) {
  tooltip.transition()
    .duration(200)
    .style("opacity", .9);
    
  tooltip.html(`Value: ${d.value}`)
    .style("left", (event.pageX + 10) + "px")
    .style("top", (event.pageY - 28) + "px");
})
.on("mouseout", function() {
  tooltip.transition()
    .duration(500)
    .style("opacity", 0);
});

// Dispatch custom events
const dispatch = d3.dispatch("select", "highlight");

dispatch.on("select", function(value) {
  console.log("Selected:", value);
});

dispatch.on("highlight", function(value) {
  console.log("Highlighted:", value);
});

// Trigger custom event
dispatch.call("select", this, "data value");

// Multiple listeners for same event
dispatch.on("select.a", callback1);
dispatch.on("select.b", callback2);

// Coordinating visualizations with dispatch
function setupCoordination(charts) {
  const dispatch = d3.dispatch("hover", "select");
  
  charts.forEach(chart => {
    chart.on = dispatch.on.bind(dispatch);
  });
  
  charts[0].on("hover", hovered => {
    charts[1].highlight(hovered);
    charts[2].highlight(hovered);
  });
  
  return dispatch;
}
```# D3.js Comprehensive Cheatsheet

## Core Concepts

### Selections
```javascript
// Select an element
d3.select("#id")              // Select element by ID
d3.select(".class")           // Select first element with class
d3.selectAll(".class")        // Select all elements with class
d3.selectAll("div")           // Select all div elements
d3.selectAll("div.myClass")   // Select all div elements with class 'myClass'
d3.selectAll("div > p")       // Select all p elements that are direct children of div

// Select by position
d3.selectAll("li:first-child") // Select first child li in each group
d3.selectAll("li:nth-child(2)") // Select second li in each group
d3.selectAll("li:last-child")  // Select last child li in each group

// Chaining selections
d3.select("body")
  .select("div")
  .selectAll("p")

// Selection methods
selection.size()              // Number of elements in selection
selection.empty()             // Returns true if selection is empty
selection.nodes()             // Returns array of all elements in selection
selection.node()              // Returns first element in selection
selection.filter(function)    // Filter selection based on data or index

// Traversal methods
selection.select(selector)    // Select first descendant matching selector
selection.selectAll(selector) // Select all descendants matching selector
selection.selectChild(selector) // Select first child matching selector (D3 v6+)
selection.selectChildren(selector) // Select all children matching selector (D3 v6+)
selection.selectParent(selector) // Select parent element (D3 v6+)
selection.ancestors()         // Select ancestors in the DOM
selection.descendants()       // Select all descendants in the DOM

// Data join
selection.data(dataArray)     // Bind data to selection
selection.data(dataArray, d => d.id) // Bind with key function
selection.datum(value)        // Bind single value to selection (no enter/exit)
selection.enter()             // Elements to be added
selection.exit()              // Elements to be removed
selection.join()              // Simplified enter/update/exit pattern
```

### Manipulating DOM
```javascript
// Setting attributes and properties
selection.attr("name", value)   // Set attribute
selection.attr("name")          // Get attribute
selection.attr("name", d => d.value) // Set attribute using data
selection.attr({                // Set multiple attributes (D3 v3 only)
  width: 100, 
  height: 200
})

selection.style("name", value)  // Set CSS property
selection.style("name")         // Get CSS property
selection.style("name", d => d.color) // Set CSS using data
selection.style({               // Set multiple CSS properties (D3 v3 only)
  color: "blue",
  "font-size": "12px"
})

selection.property("name", value) // Set DOM property
selection.property("name")      // Get DOM property
selection.property("checked", true) // Set checkbox/radio properties

selection.classed("name", boolean) // Add/remove class
selection.classed("name")       // Check if has class
selection.classed({             // Set multiple classes
  active: true,
  hidden: false
})
selection.classed("name", d => d.active) // Set class using data

selection.text("text")          // Set text content
selection.text()                // Get text content
selection.text(d => d.name)     // Set text using data

selection.html("<span>html</span>") // Set HTML content
selection.html()                // Get HTML content
selection.html(d => `<span>${d.name}</span>`) // Set HTML using data

// Appending, inserting and removing elements
selection.append("tagName")     // Append element
selection.append(d => d.element) // Append element using data

selection.insert("tagName")     // Insert element as last child
selection.insert("tagName", "before") // Insert element before selector
selection.insert(d => d.element, ":first-child") // Insert using data

selection.remove()              // Remove elements
selection.raise()               // Move element to end (top of z-stack)
selection.lower()               // Move element to beginning (bottom of z-stack)

// Order manipulation
selection.sort((a, b) => a.value - b.value) // Sort elements by data

// Dispatching events
selection.dispatch("click")     // Dispatch event
selection.dispatch("input", {bubbles: true}) // With options
```

### Data Binding
```javascript
// Basic data binding
selection.data(dataArray)                  // Bind data
selection.data(dataArray, d => d.id)       // Bind with key function
selection.data()                           // Get bound data
selection.datum(value)                     // Bind single value without enter/exit
selection.datum()                          // Get bound datum
selection.datum(d => d.value)              // Set datum using function

// Access data in callbacks
selection.attr("r", d => d.radius)         // d is the datum
selection.attr("r", (d, i) => i * 5)       // i is the index
selection.attr("r", (d, i, nodes) => {     // nodes is the selection
  return d3.select(nodes[i]).attr("cx");
})

// Enter-Update-Exit pattern in detail
const update = selection.data(dataArray);  // Update selection (existing elements)
  
const enter = update.enter()               // Enter selection (new elements)
  .append("div")                           // Create elements for entering data
  .attr("class", "new")                    // Set attributes for new elements
  .text(d => d.name);                      // Set text for new elements
  
const exit = update.exit()                 // Exit selection (elements to remove)
  .style("opacity", 0)                     // Fade out
  .remove();                               // Remove elements from DOM

// Merge enter and update selections
update.attr("class", "updated");           // Update existing elements
enter.merge(update)                        // Merge enter and update
  .style("color", d => d.color)            // Apply to both enter and update
  .text(d => d.name);                      // Set text for all elements

// Complete Enter-Update-Exit with transition
function updateChart(data) {
  // Bind data
  const bars = svg.selectAll(".bar")
    .data(data, d => d.id);
    
  // EXIT - Remove elements not in new data
  bars.exit()
    .transition()
    .duration(300)
    .style("opacity", 0)
    .remove();
    
  // ENTER - Create elements for new data
  const barsEnter = bars.enter()
    .append("rect")
    .attr("class", "bar")
    .attr("x", d => x(d.name))
    .attr("y", height)
    .attr("width", x.bandwidth())
    .attr("height", 0)
    .style("opacity", 0);
    
  // UPDATE+ENTER - Apply operations to all elements
  barsEnter.merge(bars)
    .transition()
    .duration(500)
    .attr("x", d => x(d.name))
    .attr("y", d => y(d.value))
    .attr("width", x.bandwidth())
    .attr("height", d => height - y(d.value))
    .style("opacity", 1);
}

// Join pattern (D3 v5+)
selection.join(
  enter => enter.append("div"),            // Enter selection
  update => update.style("color", "blue"), // Update selection
  exit => exit.remove()                    // Exit selection
);

// Simple join pattern
selection.join("div")                      // Shorthand for common case
  .text(d => d.name)
  .style("color", d => d.color);

// Join with key function
selection.join("div", ".key")              // Join with key
  .text(d => d.name);
```

## Scales and Axes

### Understanding Scales
Scales are functions that map from an input domain to an output range.

- **Domain**: The set of input values (data space)
- **Range**: The set of output values (usually pixel space)

```javascript
// Basic scale example
const scale = d3.scaleLinear()
  .domain([0, 100])    // Data values between 0 and 100
  .range([0, 500]);    // Map to pixels between 0 and 500

scale(0);   // Returns 0
scale(50);  // Returns 250
scale(100); // Returns 500
```

### Continuous Scales
```javascript
// Linear scale (numeric)
const linearScale = d3.scaleLinear()
  .domain([0, 100])                        // Input range
  .range([0, 500]);                        // Output range

// Using linear scale with multiple segments
const piecewiseScale = d3.scaleLinear()
  .domain([0, 50, 100])                    // Multiple domain segments
  .range([0, 300, 500]);                   // Corresponding range segments

// Clamping out-of-bounds values
const clampedScale = d3.scaleLinear()
  .domain([0, 100])
  .range([0, 500])
  .clamp(true);                            // Clamp values outside domain

clampedScale(-10);                         // Returns 0 instead of -50
clampedScale(200);                         // Returns 500 instead of 1000

// Nice domain rounding to clean values
const niceScale = d3.scaleLinear()
  .domain([0.371, 99.843])
  .nice()                                  // Rounds domain to [0, 100]
  .range([0, 500]);

// Time scale - for dates and times
const timeScale = d3.scaleTime()
  .domain([new Date(2020, 0, 1), new Date(2020, 11, 31)])
  .range([0, 800]);

timeScale(new Date(2020, 6, 1));           // Returns 400 (middle of year)

// Time scale with nice rounding 
const niceTimeScale = d3.scaleTime()
  .domain([new Date(2020, 2, 14), new Date(2020, 10, 8)])
  .nice(d3.timeMonth)                      // Rounds to months
  .range([0, 800]);                        // Domain becomes [Mar 1, Nov 30]

// Logarithmic scale - for exponential data
const logScale = d3.scaleLog()
  .domain([1, 1000])                       // Domain must be strictly positive
  .range([0, 500]);

logScale(10);                              // Returns ~166.67
logScale(100);                             // Returns ~333.33

// Log scale with custom base
const log10 = d3.scaleLog()
  .base(10)                                // Change the logarithm base
  .domain([1, 1000])
  .range([0, 500]);

// Power scale - for polynomial relationships
const sqrtScale = d3.scalePow()
  .exponent(0.5)                           // Square root scale
  .domain([0, 100])
  .range([0, 500]);

const squareScale = d3.scalePow()
  .exponent(2)                             // Square scale
  .domain([0, 10])
  .range([0, 500]);

// Radial scale - maps to radius
const radialScale = d3.scaleRadial()
  .domain([0, 100])
  .range([0, 250]);                        // Maps to radius values

// Symmetrical log scale (for data crossing zero)
const symlogScale = d3.scaleSymlog()       // Handles positive and negative values
  .domain([-1000, 1000])
  .constant(10)                            // Controls behavior near zero
  .range([0, 500]);
```

### Discrete Scales
```javascript
// Ordinal scale - maps discrete domain to discrete range
const ordinalScale = d3.scaleOrdinal()     // For categories
  .domain(["A", "B", "C"])
  .range(["red", "green", "blue"]);

ordinalScale("A");                         // Returns "red"
ordinalScale("D");                         // Returns "red" (cycles through range)

// With unknown value handling
const ordinalWithUnknown = d3.scaleOrdinal()
  .domain(["A", "B", "C"])
  .range(["red", "green", "blue"])
  .unknown("gray");                        // Color for values not in domain

ordinalWithUnknown("D");                   // Returns "gray"

// Band scale - for bar charts and columns
const bandScale = d3.scaleBand()
  .domain(["A", "B", "C", "D", "E"])
  .range([0, 500])
  .padding(0.1);                           // Padding between bands (0-1)

bandScale("A");                            // Returns starting position of band A
bandScale.bandwidth();                     // Returns width of each band

// Extra band configuration
const bandScaleDetailed = d3.scaleBand()
  .domain(["A", "B", "C"])
  .range([0, 500])
  .paddingInner(0.1)                       // Padding between bands
  .paddingOuter(0.2)                       // Padding at ends
  .align(0.5);                             // Alignment within range (0-1)

// Point scale - for scatter plots (no width)
const pointScale = d3.scalePoint()
  .domain(["A", "B", "C", "D", "E"])
  .range([0, 500])
  .padding(10);                            // Padding in pixels

pointScale("A");                           // Returns point position for "A"
pointScale.step();                         // Returns distance between points

// Threshold scale - bin continuous values
const thresholdScale = d3.scaleThreshold()
  .domain([0, 50, 100])                    // Threshold values
  .range(["cold", "mild", "hot", "extreme"]); // One more range value than domain

thresholdScale(-10);                       // Returns "cold"
thresholdScale(30);                        // Returns "mild"
thresholdScale(200);                       // Returns "extreme"

// Quantize scale - equal-sized continuous segments
const quantizeScale = d3.scaleQuantize()
  .domain([0, 100])                        // Continuous input domain
  .range(["small", "medium", "large"]);    // Discrete output range

quantizeScale(10);                         // Returns "small"
quantizeScale(50);                         // Returns "medium"
quantizeScale(90);                         // Returns "large"

// Quantile scale - based on sample distribution
const quantileScale = d3.scaleQuantile()
  .domain([3, 6, 7, 8, 8, 10, 13, 15, 16, 20, 25, 27])
  .range(["low", "medium", "high"]);       // Splits into 3 equal-count groups

// Ordinal with custom sorting
const sortedScale = d3.scaleOrdinal()
  .domain(["C", "A", "B"].sort())          // Sort domain values
  .range(["red", "green", "blue"]);
```

### Color Scales
```javascript
// Sequential color scales - for continuous data with single trend
const sequentialScale = d3.scaleSequential()
  .domain([0, 100])
  .interpolator(d3.interpolateViridis);    // Cool to warm colors

// Built-in color interpolators
const plasmaScale = d3.scaleSequential(d3.interpolatePlasma)
  .domain([0, 100]);

const infernScale = d3.scaleSequential(d3.interpolateInferno)
  .domain([0, 100]);

const magmaScale = d3.scaleSequential(d3.interpolateMagma)
  .domain([0, 100]);

const warmScale = d3.scaleSequential(d3.interpolateWarm)
  .domain([0, 100]);

const coolScale = d3.scaleSequential(d3.interpolateCool)
  .domain([0, 100]);

const rainbowScale = d3.scaleSequential(d3.interpolateRainbow)
  .domain([0, 100]);

const cubehelixScale = d3.scaleSequential(d3.interpolateCubehelixDefault)
  .domain([0, 100]);

// Custom sequential scale with interpolateHsl
const blueToRed = d3.scaleSequential()
  .domain([0, 100])
  .interpolator(t => d3.interpolateHsl("blue", "red")(t));

// Diverging color scales - for data with meaningful midpoint
const divergingScale = d3.scaleDiverging()
  .domain([-100, 0, 100])                  // [min, mid, max]
  .interpolator(d3.interpolateRdBu);       // Red-Blue diverging colors

// Common diverging color schemes
const rdYlBuScale = d3.scaleDiverging(d3.interpolateRdYlBu)
  .domain([-100, 0, 100]);

const spectralScale = d3.scaleDiverging(d3.interpolateSpectral)
  .domain([-100, 0, 100]);

// Cyclical scales - for angles, months, days, etc.
const cyclicalScale = d3.scaleSequential()
  .domain([0, 365])
  .interpolator(d3.interpolateRainbow);    // Colors cycle through rainbow

// Quantized color scales
const quantizedColScale = d3.scaleQuantize()
  .domain([0, 100])
  .range(d3.schemeBlues[5]);               // 5 blue shades 

// Ordinal color scales with predefined schemes
const categoryScale = d3.scaleOrdinal()
  .domain(["A", "B", "C", "D", "E", "F", "G"])
  .range(d3.schemeCategory10);             // 10 categorical colors

// Other categorical schemes
const pastelScale = d3.scaleOrdinal(d3.schemePastel1); // Pastel colors
const setScale = d3.scaleOrdinal(d3.schemeSet1);       // Set1 colors
const tableauScale = d3.scaleOrdinal(d3.schemeTableau10); // Tableau colors
```

### Scale Methods and Utilities
```javascript
// Inverting scales (range → domain)
const scale = d3.scaleLinear().domain([0, 100]).range([0, 500]);
scale.invert(250);                         // Returns 50

// Check if value is in domain
const xScale = d3.scaleLinear().domain([0, 100]).range([0, 500]);
xScale.clamp(true);                        // Limit values to range
xScale(-10);                               // Returns 0

// Copy scales
const scale1 = d3.scaleLinear().domain([0, 100]).range([0, 500]);
const scale2 = scale1.copy();              // Creates independent copy

// Ticks generation
const tickScale = d3.scaleLinear().domain([0, 100]);
tickScale.ticks(5);                        // Returns [0, 20, 40, 60, 80, 100]
tickScale.ticks(10);                       // Returns [0, 10, 20, 30, ... 100]

// Custom tick format
const formatScale = d3.scaleLinear().domain([0, 100]).range([0, 500]);
const tickFormat = formatScale.tickFormat(5, "+.1f"); // Format with 1 decimal
tickFormat(50);                            // Returns "+50.0"

// Getting scale's domain and range
scale.domain();                            // Returns current domain
scale.range();                             // Returns current range

// Testing if a function is a scale
// No built-in method, but can check for .domain and .range methods
function isScale(obj) {
  return typeof obj === 'function' && 
         typeof obj.domain === 'function' && 
         typeof obj.range === 'function';
}
```

### Axes
```javascript
// Creating axes
const xAxis = d3.axisBottom(xScale);       // X-axis below
const yAxis = d3.axisLeft(yScale);         // Y-axis to the left
const topAxis = d3.axisTop(xScale);        // X-axis above
const rightAxis = d3.axisRight(yScale);    // Y-axis to the right

// Customizing axes
xAxis.ticks(5)                             // Number of ticks
  .tickFormat(d3.format(".0f"))            // Tick format
  .tickSize(6)                             // Tick size
  .tickSizeInner(6)                        // Inner tick size
  .tickSizeOuter(0)                        // Outer tick size
  .tickPadding(8);                         // Padding between tick and label

// Tick arguments
yAxis.ticks(10);                           // Approximate number of ticks
yAxis.ticks(d3.timeMinute.every(15));      // Time intervals (every 15 min)
yAxis.ticks(5, "s");                       // 5 ticks with SI units

// Tick formats
xAxis.tickFormat(d3.format(".0%"));        // Percentage format
xAxis.tickFormat(d3.format("$.2f"));       // Currency format
xAxis.tickFormat(d3.format("+,d"));        // Signed format with commas
xAxis.tickFormat(d3.format(".2s"));        // SI-prefix with 2 significant digits
xAxis.tickFormat(d3.timeFormat("%b %d"));  // Date format (Apr 30)
xAxis.tickFormat(null);                    // Use default format

// Format strings
// d - decimal integer
// f - float
// s - SI-prefix
// % - percentage
// p - multiply by 100, format as percentage
// e - exponent
// g - decimal or exponent, depending on value
// , - group thousands
// r - decimal rounded to significant digits

// Time formats
// %Y - year (e.g., 2024)
// %B - full month name (e.g., April)
// %b - abbreviated month (e.g., Apr)
// %d - zero-padded day of month (e.g., 01)
// %H - hour (00-23)
// %M - minute (00-59)
// %S - second (00-59)
// %L - milliseconds (000-999)
// %A - full weekday name (e.g., Monday)
// %a - abbreviated weekday (e.g., Mon)

// Custom tick values
xAxis.tickValues([0, 25, 50, 75, 100]);    // Explicitly set tick values

// Rendering axes
svg.append("g")
  .attr("class", "x-axis")                 // Add a class for styling
  .attr("transform", "translate(0, 300)")  // Position the axis
  .call(xAxis);                            // Call the axis generator

svg.append("g")
  .attr("class", "y-axis")
  .call(yAxis);

// Styling axes with CSS
// .x-axis path, .x-axis line {
//   stroke: #ccc;
//   stroke-width: 1;
// }
// .x-axis text {
//   font-size: 12px;
//   font-family: sans-serif;
// }

// Styling axes with D3
svg.selectAll(".x-axis path, .x-axis line")
  .style("stroke", "#ccc")
  .style("stroke-width", 1);

svg.selectAll(".x-axis text")
  .style("font-size", "12px")
  .style("font-family", "sans-serif");

// Add a title to the axis
svg.select(".y-axis")
  .append("text")
  .attr("transform", "rotate(-90)")
  .attr("y", -40)
  .attr("x", -150)
  .attr("text-anchor", "middle")
  .attr("fill", "black")
  .text("Value (units)");

// Grid lines using axis generators
function makeXGridlines() {
  return d3.axisBottom(xScale)
    .ticks(5);
}

svg.append("g")
  .attr("class", "grid")
  .attr("transform", "translate(0," + height + ")")
  .call(makeXGridlines()
    .tickSize(-height)
    .tickFormat("")
  );

// Transitioning axes
svg.select(".x-axis")
  .transition()
  .duration(750)
  .call(xAxis);

// Updating axes on data change
function updateAxes(newData) {
  // Update scale domains
  xScale.domain([0, d3.max(newData, d => d.x)]);
  yScale.domain([0, d3.max(newData, d => d.y)]);
  
  // Transition axes
  svg.select(".x-axis")
    .transition()
    .duration(500)
    .call(xAxis);
    
  svg.select(".y-axis")
    .transition()
    .duration(500)
    .call(yAxis);
}

// Responsive axes on window resize
function resize() {
  const width = parseInt(container.style("width")) - margin.left - margin.right;
  
  // Update scale range
  xScale.range([0, width]);
  
  // Update axes
  svg.select(".x-axis").call(xAxis);
}

window.addEventListener("resize", resize);
```

## Shapes and Layouts

### Shapes (Generators)
```javascript
// Line generator
const line = d3.line()                     // Create line generator
  .x(d => xScale(d.x))                     // X-coordinate accessor
  .y(d => yScale(d.y))                     // Y-coordinate accessor
  .curve(d3.curveLinear);                  // Linear interpolation

// Line generator usage
svg.append("path")
  .datum(data)                             // Bind single array of data
  .attr("d", line)                         // Use line generator on data
  .attr("fill", "none")                    // Don't fill line
  .attr("stroke", "steelblue")             // Line color
  .attr("stroke-width", 1.5);              // Line width

// Curve interpolation methods
line.curve(d3.curveLinear);                // Straight line segments (default)
line.curve(d3.curveStep);                  // Step segments, midpoint
line.curve(d3.curveStepBefore);            // Step segments, before
line.curve(d3.curveStepAfter);             // Step segments, after
line.curve(d3.curveBasis);                 // B-spline
line.curve(d3.curveCardinal);              // Cardinal spline
line.curve(d3.curveMonotoneX);             // Monotone cubic, preserves monotonicity
line.curve(d3.curveCatmullRom);            // Cubic spline
line.curve(d3.curveNatural);               // Natural cubic spline

// Tension configuration for curves
line.curve(d3.curveCardinal.tension(0.5)); // Cardinal with tension
line.curve(d3.curveCatmullRom.alpha(0.5)); // Catmull-Rom with alpha

// Area generator
const area = d3.area()
  .x(d => xScale(d.x))                     // X-coordinate accessor
  .y0(height)                              // Baseline Y-coordinate
  .y1(d => yScale(d.y))                    // Upper Y-coordinate
  .curve(d3.curveLinear);                  // Linear interpolation

// Area generator usage
svg.append("path")
  .datum(data)
  .attr("d", area)
  .attr("fill", "steelblue")
  .attr("fill-opacity", 0.3)
  .attr("stroke", "steelblue")
  .attr("stroke-width", 1.5);

// Stacked area
const stackedArea = d3.area()
  .x(d => xScale(d.date))
  .y0(d => yScale(d[0]))                   // Bottom of area (from stack)
  .y1(d => yScale(d[1]));                  // Top of area (from stack)

// Horizontal area
const horizontalArea = d3.area()
  .y(d => yScale(d.y))                     // Y-coordinate accessor
  .x0(0)                                   // Baseline X-coordinate
  .x1(d => xScale(d.x));                   // Right X-coordinate

// Radial area
const radialArea = d3.areaRadial()
  .angle(d => angleScale(d.angle))         // Angle accessor
  .innerRadius(d => 0)                     // Inner radius
  .outerRadius(d => radiusScale(d.radius)); // Outer radius

// Arc generator (pie/donut charts)
const arc = d3.arc()
  .innerRadius(0)                          // 0 for pie, >0 for donut
  .outerRadius(100)                        // Outer radius
  .startAngle(d => d.startAngle)           // Start angle (from pie layout)
  .endAngle(d => d.endAngle)               // End angle (from pie layout)
  .padAngle(0.02)                          // Padding between arcs
  .padRadius(100)                          // Radius at which pad is applied
  .cornerRadius(4);                        // Rounded corners

// Arc generator usage (with pie layout)
const pie = d3.pie()
  .value(d => d.value)
  .sort(null);                             // Don't sort by value

const arcs = pie(data);                    // Generate arc data

svg.selectAll("path")
  .data(arcs)
  .join("path")
  .attr("d", arc)
  .attr("fill", d => colorScale(d.data.name))
  .attr("stroke", "white");

// Arc transitions
svg.selectAll("path")
  .data(updatedArcs)
  .transition()
  .duration(750)
  .attrTween("d", function(d) {
    const interpolate = d3.interpolate(this._current, d);
    this._current = interpolate(0);
    return function(t) {
      return arc(interpolate(t));
    };
  });

// Symbol generator (for scatter plots)
const symbol = d3.symbol()
  .type(d3.symbolCircle)                   // Symbol type
  .size(64);                               // Symbol size in square pixels

// Symbol types
symbol.type(d3.symbolCircle);              // Circle
symbol.type(d3.symbolCross);               // Cross
symbol.type(d3.symbolDiamond);             // Diamond
symbol.type(d3.symbolSquare);              // Square
symbol.type(d3.symbolStar);                // Star
symbol.type(d3.symbolTriangle);            // Triangle
symbol.type(d3.symbolWye);                 // Y shape
symbol.type(d => d.type);                  // Dynamic based on data

// Symbol usage
svg.selectAll("path")
  .data(data)
  .join("path")
  .attr("d", symbol)
  .attr("transform", d => `translate(${xScale(d.x)},${yScale(d.y)})`)
  .attr("fill", "steelblue");

// Stack generator (for stacked charts)
const stack = d3.stack()
  .keys(["apples", "oranges", "bananas"])  // Stack layers by keys
  .order(d3.stackOrderNone)                // Stack order
  .offset(d3.stackOffsetNone);             // Stack offset

// Stack ordering
stack.order(d3.stackOrderNone);            // No reordering (default)
stack.order(d3.stackOrderAscending);       // Smallest values on bottom
stack.order(d3.stackOrderDescending);      // Largest values on bottom
stack.order(d3.stackOrderInsideOut);       // Largest values in middle
stack.order(d3.stackOrderReverse);         // Reverse natural order

// Stack offset (baseline position)
stack.offset(d3.stackOffsetNone);          // Zero baseline (default)
stack.offset(d3.stackOffsetExpand);        // Normalize to 100%
stack.offset(d3.stackOffsetDiverging);     // Positive/negative values
stack.offset(d3.stackOffsetSilhouette);    // Centered around y=0
stack.offset(d3.stackOffsetWiggle);        // Minimize wiggle (stream graphs)

// Stack usage
const stackedData = stack(data);           // Returns array of series

// Chord generator (for chord diagrams)
const chord = d3.chord()
  .padAngle(0.05)                          // Padding between groups
  .sortSubgroups(d3.descending)            // Sort relationships
  .sortChords(d3.descending);              // Sort chords

// Link generators
const linkHorizontal = d3.linkHorizontal()
  .x(d => d.x)
  .y(d => d.y);

const linkVertical = d3.linkVertical()
  .x(d => d.x)
  .y(d => d.y);

const linkRadial = d3.linkRadial()
  .angle(d => d.angle)
  .radius(d => d.radius);

// Custom curve generator
const myCurve = d3.line()
  .x(d => xScale(d.x))
  .y(d => yScale(d.y))
  .curve(d3.curveBasis);                   // Basis spline curve
```

### Layouts
```javascript
// Hierarchy layout (for trees, treemaps, etc.)
const root = d3.hierarchy(data)            // Convert hierarchical data
  .sum(d => d.value)                       // Sum values for sizing
  .sort((a, b) => b.value - a.value);      // Sort by value

// Hierarchy accessor methods
root.descendants();                         // Array of all nodes
root.ancestors();                           // Array of node's ancestors
root.links();                               // Array of parent-child links
root.path(targetNode);                      // Path from node to target
root.leaves();                              // Array of leaf nodes
root.find(d => d.name === "target");        // Find specific node

// Tree layout
const tree = d3.tree()                     // Create tree layout
  .size([width, height])                   // Overall size
  .separation((a, b) => a.parent === b.parent ? 1 : 2); // Node separation

const treeRoot = tree(root);               // Apply layout to hierarchy

// Drawing tree
svg.selectAll("path")
  .data(treeRoot.links())                  // Links between nodes
  .join("path")
  .attr("d", d3.linkVertical()
    .x(d => d.x)
    .y(d => d.y));

svg.selectAll("circle")
  .data(treeRoot.descendants())            // All nodes
  .join("circle")
  .attr("cx", d => d.x)
  .attr("cy", d => d.y)
  .attr("r", 5);

// Cluster layout (dendrogram)
const cluster = d3.cluster()
  .size([width, height]);

const clusterRoot = cluster(root);

// Treemap layout
const treemap = d3.treemap()
  .size([width, height])
  .padding(2)
  .round(true);

const treemapRoot = treemap(root);

// Drawing treemap
svg.selectAll("rect")
  .data(treemapRoot.leaves())
  .join("rect")
  .attr("x", d => d.x0)
  .attr("y", d => d.y0)
  .attr("width", d => d.x1 - d.x0)
  .attr("height", d => d.y1 - d.y0)
  .attr("fill", d => colorScale(d.data.category));

// Treemap tiling algorithms
treemap.tile(d3.treemapBinary);            // Binary subdivision
treemap.tile(d3.treemapSquarify);          // Squarified (default)
treemap.tile(d3.treemapSlice);             // Horizontal slices
treemap.tile(d3.treemapDice);              // Vertical slices
treemap.tile(d3.treemapSliceDice);         // Alternate slice and dice

// Partition layout (sunburst, icicle)
const partition = d3.partition()
  .size([2 * Math.PI, radius * radius])
  .padding(2);

const partitionRoot = partition(root);

// Drawing sunburst
svg.selectAll("path")
  .data(partitionRoot.descendants().slice(1))
  .join("path")
  .attr("d", d3.arc()
    .startAngle(d => d.x0)
    .endAngle(d => d.x1)
    .innerRadius(d => Math.sqrt(d.y0))
    .outerRadius(d => Math.sqrt(d.y1)))
  .attr("fill", d => colorScale(d.data.category));

// Pack layout (circle packing)
const pack = d3.pack()
  .size([width, height])
  .padding(3);

const packRoot = pack(root);

// Drawing packed circles
svg.selectAll("circle")
  .data(packRoot.descendants())
  .join("circle")
  .attr("cx", d => d.x)
  .attr("cy", d => d.y)
  .attr("r", d => d.r)
  .attr("fill", d => d.children ? "lightgray" : "steelblue");

// Force simulation (for force-directed graphs)
const simulation = d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links).id(d => d.id))
  .force("charge", d3.forceManyBody().strength(-50))
  .force("center", d3.forceCenter(width / 2, height / 2))
  .on("tick", ticked);

// Force configuration
simulation
  .alphaTarget(0.3)                        // Target velocity 
  .alphaDecay(0.05)                        // Decay rate
  .alphaMin(0.001);                        // Stop threshold

// Specific forces
const linkForce = d3.forceLink(links)
  .id(d => d.id)                           // How to determine node identity
  .distance(50)                            // Target link distance
  .strength(0.9);                          // Link strength

const chargeForce = d3.forceManyBody()
  .strength(-30)                           // Negative is repulsion
  .theta(0.9)                              // Barnes-Hut approximation
  .distanceMin(1)                          // Minimum distance constraint
  .distanceMax(Infinity);                  // Maximum distance constraint

// Additional forces
simulation.force("x", d3.forceX(width / 2)); // Centering force in x direction
simulation.force("y", d3.forceY(height / 2)); // Centering force in y direction

simulation.force("collision", d3.forceCollide(d => d.radius + 1)); // Prevent overlap

// Force directed graph tick function
function ticked() {
  link
    .attr("x1", d => d.source.x)
    .attr("y1", d => d.source.y)
    .attr("x2", d => d.target.x)
    .attr("y2", d => d.target.y);

  node
    .attr("cx", d => d.x)
    .attr("cy", d => d.y);
}

// Start, stop, and restart simulation
simulation.stop();                        // Stop simulation
simulation.restart();                     // Restart simulation 
simulation.alpha(1).restart();            // Reset alpha and restart

// Manually tick simulation for static layout
simulation.tick(300);                     // Run 300 ticks without animation

// Histogram
const histogram = d3.histogram()
  .value(d => d.value)                    // Value accessor
  .domain(xScale.domain())                // Data range
  .thresholds(xScale.ticks(20));          // Bin count or specific thresholds
  
const bins = histogram(data);             // Generate bins

// Drawing histogram
svg.selectAll("rect")
  .data(bins)
  .join("rect")
  .attr("x", d => xScale(d.x0))
  .attr("y", d => yScale(d.length))
  .attr("width", d => xScale(d.x1) - xScale(d.x0) - 1)
  .attr("height", d => height - yScale(d.length))
  .attr("fill", "steelblue");

// Contour (for density maps)
const contours = d3.contourDensity()
  .x(d => xScale(d.x))                    // X-position accessor
  .y(d => yScale(d.y))                    // Y-position accessor
  .size([width, height])                  // Output size
  .bandwidth(30)                          // Kernel size
  .thresholds(10);                        // Number of contours

// Generate contours
const contourData = contours(data);

// Draw contours
svg.selectAll("path")
  .data(contourData)
  .join("path")
  .attr("d", d3.geoPath())
  .attr("fill", d => colorScale(d.value))
  .attr("stroke", "black")
  .attr("stroke-opacity", 0.2);

// Voronoi diagram
const delaunay = d3.Delaunay
  .from(data, d => xScale(d.x), d => yScale(d.y));
const voronoi = delaunay.voronoi([0, 0, width, height]);

// Draw Voronoi cells
svg.selectAll("path")
  .data(data)
  .join("path")
  .attr("d", (d, i) => voronoi.renderCell(i))
  .attr("fill", "none")
  .attr("stroke", "black");

// Finding nearest point with Delaunay triangulation
const point = delaunay.find(mouseX, mouseY);

// Sankey diagram
const sankey = d3.sankey()
  .nodeWidth(15)                          // Width of node
  .nodePadding(10)                        // Vertical padding between nodes
  .extent([[1, 1], [width - 1, height - 5]]); // Layout extent

// First pass to compute layout
const { nodes, links } = sankey({
  nodes: data.nodes.map(d => Object.assign({}, d)),
  links: data.links.map(d => Object.assign({}, d))
});

// Chord diagram
const chord = d3.chord()
  .padAngle(0.05)                         // Padding between groups
  .sortSubgroups(d3.descending)           // Sort relationships
  .sortChords(d3.descending);             // Sort chords

const chords = chord(matrix);             // Generate chord data

// Drawing chord groups
svg.selectAll("path")
  .data(chords.groups)
  .join("path")
  .attr("d", d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(outerRadius))
  .attr("fill", d => colorScale(d.index));

// Drawing chord ribbons
svg.selectAll("path")
  .data(chords)
  .join("path")
  .attr("d", d3.ribbon()
    .radius(innerRadius))
  .attr("fill", d => colorScale(d.target.index))
  .attr("fill-opacity", 0.7);
```

## Transitions and Interaction

### Transitions
```javascript
// Basic transition
selection.transition()                     // Create transition
  .duration(1000)                          // Duration in ms
  .delay(500)                              // Delay before starting
  .ease(d3.easeLinear)                     // Easing function
  .attr("cx", d => xScale(d.x))            // Animate attribute
  .style("fill", "blue");                  // Animate style

// Named transition for control
selection.transition("fade")               // Name the transition
  .duration(750);

// Transition events
selection.transition()
  .on("start", function() {                // Transition start
    d3.select(this).style("fill", "red");
  })
  .on("end", function() {                  // Transition end
    d3.select(this).style("fill", "blue");
  })
  .on("interrupt", function() {            // Transition interrupted
    console.log("Interrupted");
  });

// Transition delay patterns
selection.transition()
  .delay((d, i) => i * 50)                 // Sequential delay by index
  .duration(500);

// Chained transitions
selection.transition()
  .duration(1000)
  .attr("r", 10)
  .transition()                            // Chain another transition
  .duration(1000)
  .attr("r", 5);

// Transition selection methods
const t = selection.transition();
t.selection();                             // Get the original selection

// Interrupt transitions
selection.interrupt();                     // Interrupt all transitions
selection.interrupt("fade");               // Interrupt named transition

// Easing functions
selection.transition().ease(d3.easeLinear);    // Linear easing (default)
selection.transition().ease(d3.easeQuad);      // Quadratic easing
selection.transition().ease(d3.easeCubic);     // Cubic easing
selection.transition().ease(d3.easeSin);       // Sinusoidal easing
selection.transition().ease(d3.easeExp);       // Exponential easing
selection.transition().ease(d3.easeCircle);    // Circular easing
selection.transition().ease(d3.easeElastic);   // Elastic easing
selection.transition().ease(d3.easeBack);      // Back easing
selection.transition().ease(d3.easeBounce);    // Bounce easing

// Easing variations
selection.transition().ease(d3.easeQuadIn);    // Ease in
selection.transition().ease(d3.easeQuadOut);   // Ease out
selection.transition().ease(d3.easeQuadInOut); // Ease in and out

// Custom ease function
selection.transition().ease(t => t * t);       // Custom easing 

// Tweening
selection.transition()
  .attrTween("d", function(d) {
    const interpolate = d3.interpolate(oldPath, newPath);
    return function(t) {
      return interpolate(t);
    };
  });

// Custom tweening for path
path.transition()
  .duration(750)
  .attrTween("d", function() {
    const previous = d3.select(this).attr("d");
    const current = line(newData);
    return d3.interpolatePath(previous, current);
  });

// Style and attribute tweens
selection.transition()
  .styleTween("fill", function() {         // Tween a style
    return d3.interpolateRgb("red", "blue");
  })
  .attrTween("r", function(d) {            // Tween an attribute
    const i = d3.interpolateNumber(0, d.radius);
    return function(t) { return i(t); };
  });

// Transition timing
d3.timeout(callback, delay);               // Execute after delay
d3.interval(callback, delay);              // Execute every delay

// Custom transition 
d3.transition()
  .duration(750)
  .ease(d3.easeCubic);

// Applying transitions to selections
svg.transition(t)                          // Use defined transition
  .style("opacity", 0);

// Staggered transitions
selection.each(function(d, i) {
  d3.select(this)
    .transition()
    .delay(i * 100)
    .attr("r", 10);
});

// Transition function for path elements
function arcTween(d) {
  const interpolate = d3.interpolate(this._current, d);
  this._current = interpolate(0);
  return function(t) {
    return arc(interpolate(t));
  };
}

svg.selectAll("path")
  .data(newArcs)
  .transition()
  .duration(750)
  .attrTween("d", arcTween);

// Text transition with counting effect
counter.transition()
  .duration(2000)
  .tween("text", function() {
    const i = d3.interpolateNumber(0, finalValue);
    return function(t) {
      this.textContent = Math.round(i(t));
    };
  });

// Transition all elements in a staggered sequence
function staggeredTransition(selection, delay = 50) {
  return selection
    .transition()
    .delay((d, i) => i * delay)
    .duration(500);
}

staggeredTransition(svg.selectAll("circle"))
  .attr("r", d => d.radius);
```

### Interaction
```javascript
// Event handling
selection.on("mouseover", function(event, d) {
  d3.select(this).attr("fill", "red");
  // 'this' refers to DOM element, d is the data
});

// Dragging
const drag = d3.drag()
  .on("start", dragstarted)
  .on("drag", dragged)
  .on("end", dragended);

selection.call(drag);                      // Apply drag behavior

// Zooming
const zoom = d3.zoom()
  .scaleExtent([1, 10])                    // Min/max zoom scale
  .on("zoom", zoomed);

function zoomed(event) {
  svg.attr("transform", event.transform);  // Apply zoom transform
}

svg.call(zoom);                            // Apply zoom behavior
```

## Utility Functions

### Array and Math
```javascript
// Array statistics
d3.min(array)                              // Minimum value
d3.min(array, accessor)                    // Min with accessor function
d3.max(array)                              // Maximum value
d3.max(array, accessor)                    // Max with accessor function
d3.extent(array)                           // [min, max]
d3.extent(array, accessor)                 // [min, max] with accessor
d3.sum(array)                              // Sum of values
d3.sum(array, accessor)                    // Sum with accessor function
d3.mean(array)                             // Mean value
d3.mean(array, accessor)                   // Mean with accessor function
d3.median(array)                           // Median value
d3.median(array, accessor)                 // Median with accessor function
d3.quantile(array, p)                      // p-quantile value (0-1)
d3.quantileSorted(sorted, p)               // p-quantile for sorted array
d3.variance(array)                         // Sample variance
d3.deviation(array)                        // Standard deviation
d3.fsum(array)                             // More accurate sum (v7+)
d3.fcumsum(array)                          // Cumulative accurate sum (v7+)

// Array manipulation
d3.range(start, stop, step)                // Generate range of numbers
d3.range(10)                               // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
d3.range(0, 10, 2)                         // [0, 2, 4, 6, 8]

d3.shuffle(array)                          // Random permutation (in place)
d3.shuffler(random)                        // Create shuffle with custom RNG
d3.ticks(start, stop, count)               // Generate nice tick values
d3.tickIncrement(start, stop, count)       // Compute tick increment
d3.tickStep(start, stop, count)            // Compute tick step
d3.nice(start, stop, count)                // Extend domain to nice ticks
d3.zip(...arrays)                          // Zip arrays together
d3.transpose(matrix)                       // Transpose matrix
d3.pairs(array)                            // Adjacent pairs
d3.permute(array, indexes)                 // Permute array by indexes
d3.merge(arrays)                           // Merge arrays
d3.sort(array, compare)                    // Sort (stable)
d3.ascending(a, b)                         // Ascending comparator
d3.descending(a, b)                        // Descending comparator
d3.reverse(array)                          // Reverse array

// Set operations
d3.difference(a, b)                        // Set difference
d3.union(a, b)                             // Set union
d3.intersection(a, b)                      // Set intersection
d3.superset(a, b)                          // Is a a superset of b?
d3.subset(a, b)                            // Is a a subset of b?
d3.disjoint(a, b)                          // Are a and b disjoint?

// Grouping data
const grouped = d3.group(data, d => d.category)  // Group by key
const nested = d3.group(data,              // Multi-level grouping
  d => d.region,
  d => d.category
)

// Access nested groups
grouped.get("category1")                   // Get one group
nested.get("east").get("category1")        // Access nested group

// Map grouped data
const rollup = d3.rollup(data,             // Map groups to values
  v => d3.mean(v, d => d.value),           // Aggregation function
  d => d.category                          // Group by key
)

// Check grouped data 
grouped.has("category1")                   // Check if key exists
rollup.get("category1")                    // Get aggregated value

// Manipulating objects
d3.entries(object)                         // Object to key-value pairs
d3.values(object)                          // Object values as array
d3.keys(object)                            // Object keys as array
d3.map(object)                             // Object to map
d3.nest()                                  // Deprecated, use d3.group
  .key(d => d.category)
  .entries(data)

// Random number generators
const random = d3.randomUniform(0, 10);    // Uniform distribution [min, max)
random();                                  // Generate random number
d3.randomNormal(mean, deviation);          // Normal (Gaussian) distribution
d3.randomLogNormal(mean, deviation);       // Log-normal distribution
d3.randomBates(n);                         // Bates distribution
d3.randomIrwinHall(n);                     // Irwin-Hall distribution
d3.randomExponential(lambda);              // Exponential distribution
d3.randomPareto(alpha);                    // Pareto distribution
d3.randomBernoulli(p);                     // Bernoulli distribution
d3.randomGeometric(p);                     // Geometric distribution
d3.randomBinomial(n, p);                   // Binomial distribution
d3.randomGamma(k, theta);                  // Gamma distribution
d3.randomBeta(alpha, beta);                // Beta distribution
d3.randomCauchy(a, b);                     // Cauchy distribution
d3.randomLogistic(a, b);                   // Logistic distribution
d3.randomPoisson(lambda);                  // Poisson distribution
d3.randomInt(min, max);                    // Integer uniform distribution

// Number formatting
d3.format(".0f")(123.45)                   // Format as "123"
d3.format(".2f")(123.45)                   // Format as "123.45"
d3.format(".0%")(0.1234)                   // Format as "12%"
d3.format("+.0f")(123.45)                  // Format as "+123"
d3.format(",.0f")(1234.56)                 // Format as "1,235"
d3.format(".2s")(1234.56)                  // Format as "1.2k"
d3.format(".2e")(1234.56)                  // Format as "1.23e+3"
d3.format(".^20")(1234.56)                 // Fixed width
d3.formatPrefix(",.0", 1e6)(1234567)       // Format with units

// Alternative math functions
d3.bisect(array, value);                   // Binary search sorted array
d3.bisectLeft(array, value);               // Leftmost insertion point
d3.bisectRight(array, value);              // Rightmost insertion point
d3.bisector(accessor);                     // Create bisector with accessor
d3.quantize(interpolator, n);              // Create n uniformly spaced samples

// Interpolation
d3.interpolate(a, b);                      // Interpolate two values
d3.interpolateNumber(a, b);                // Interpolate two numbers
d3.interpolateRound(a, b);                 // Interpolate and round
d3.interpolateString(a, b);                // Interpolate two strings
d3.interpolateDate(a, b);                  // Interpolate two dates
d3.interpolateArray(a, b);                 // Interpolate two arrays
d3.interpolateObject(a, b);                // Interpolate two objects
d3.interpolateTransformCss(a, b);          // Interpolate CSS transforms
d3.interpolateTransformSvg(a, b);          // Interpolate SVG transforms
d3.interpolateZoom(a, b);                  // Interpolate zoom transforms
d3.interpolateRgb(a, b);                   // Interpolate RGB colors
d3.interpolateHsl(a, b);                   // Interpolate HSL colors
d3.interpolateHcl(a, b);                   // Interpolate HCL colors
d3.interpolateLab(a, b);                   // Interpolate Lab colors
d3.interpolateCubehelix(a, b);             // Interpolate Cubehelix colors

// Color space conversions
d3.rgb(r, g, b);                           // Create RGB color
d3.rgb("#ff0000");                         // Convert to RGB color
d3.hsl(h, s, l);                           // Create HSL color
d3.hsl("#ff0000");                         // Convert to HSL color
d3.lab(l, a, b);                           // Create Lab color
d3.hcl(h, c, l);                           // Create HCL color
d3.lch(l, c, h);                           // Create LCH color
d3.gray(l);                                // Create grayscale Lab color
d3.cubehelix(h, s, l);                     // Create Cubehelix color
d3.color("steelblue");                     // Create color from CSS name

// Time formatting and parsing
d3.timeFormat("%Y-%m-%d")(new Date());     // Format date as "2022-01-01"
d3.timeFormat("%B %d, %Y")(new Date());    // Format as "January 01, 2022"
d3.timeFormat("%I:%M %p")(new Date());     // Format as "10:30 AM"
d3.timeParse("%Y-%m-%d")("2022-01-01");    // Parse string to Date

// Time intervals
d3.timeDay.count(date1, date2);            // Count days between dates
d3.timeDay.floor(date);                    // Round down to day start
d3.timeDay.ceil(date);                     // Round up to day start
d3.timeDay.offset(date, count);            // Add count days to date
d3.timeDay.range(start, end, step);        // Generate dates by day

// Other time intervals
d3.timeMillisecond;                        // Millisecond interval
d3.timeSecond;                             // Second interval
d3.timeMinute;                             // Minute interval
d3.timeHour;                               // Hour interval
d3.timeDay;                                // Day interval
d3.timeWeek;                               // Week interval
d3.timeSunday;                             // Sunday-based week
d3.timeMonday;                             // Monday-based week
d3.timeMonth;                              // Month interval
d3.timeYear;                               // Year interval

// UTC time intervals (same methods as above)
d3.utcDay;                                 // UTC day interval
d3.utcWeek;                                // UTC week interval
d3.utcMonth;                               // UTC month interval
```

### Loading Data
```javascript
// Load CSV
d3.csv("data.csv").then(data => {
  // data is an array of objects with properties from CSV headers
  console.log(data);
}).catch(error => {
  console.error("Error loading data: ", error);
});

// CSV with row conversion
d3.csv("data.csv", row => {
  return {
    // Convert numeric strings to numbers
    x: +row.x,                             // Convert to number with unary +
    y: +row.y,
    category: row.category                 // Keep string as is
  };
}).then(data => {
  console.log(data);                       // Converted data
});

// Load CSV with DSV parser
const customParser = d3.dsvFormat("|");    // Custom delimiter parser
customParser.parse("a|b|c\n1|2|3");        // Parse pipe-delimited string

// Load other formats
d3.json("data.json")                       // Load JSON
  .then(data => console.log(data));

d3.xml("data.xml")                         // Load XML
  .then(data => console.log(data));

d3.html("fragment.html")                   // Load HTML
  .then(data => console.log(data));

d3.text("file.txt")                        // Load plain text
  .then(data => console.log(data));

d3.tsv("data.tsv")                         // Load TSV (tab-separated)
  .then(data => console.log(data));

d3.dsv(",", "data.csv")                    // Explicit DSV parser
  .then(data => console.log(data));

// Loading binary data
d3.blob("data.zip")                        // Load as Blob
  .then(blob => console.log(blob));

d3.buffer("data.zip")                      // Load as ArrayBuffer
  .then(buffer => console.log(buffer));

d3.image("image.png")                      // Load image
  .then(image => console.log(image));

// Multiple files with Promise
Promise.all([
  d3.csv("data1.csv"),
  d3.json("data2.json")
]).then(([csv, json]) => {
  console.log(csv, json);                  // Use both datasets
}).catch(error => {
  console.error("Error loading files:", error);
});

// Raw fetch with D3 parser
fetch("data.csv")
  .then(response => response.text())
  .then(text => d3.csvParse(text))
  .then(data => console.log(data));

// Parser functions for strings
d3.csvParse(string);                       // Parse CSV string
d3.tsvParse(string);                       // Parse TSV string
d3.csvParseRows(string);                   // Parse to array of arrays
d3.csvFormat(data);                        // Format array as CSV
d3.csvFormatRows(data);                    // Format rows as CSV

// Handle HTTP errors (custom example)
function loadData(url) {
  return d3.csv(url)
    .then(data => {
      if (data.length === 0) {
        throw new Error("No data available");
      }
      return data;
    })
    .catch(error => {
      console.error(`Failed to load ${url}: ${error.message}`);
      // Return fallback data or re-throw
      return [];
    });
}

// Streaming for large datasets
// Note: Using Fetch API streaming (no built-in D3 method)
function streamCSV(url, rowCallback, completeCallback) {
  const parser = d3.dsvFormat(",");
  let buffer = "";
  let headerRow;

  fetch(url)
    .then(response => {
      const reader = response.body.getReader();
      
      function processText({ done, value }) {
        if (done) {
          completeCallback();
          return;
        }
        
        buffer += new TextDecoder().decode(value);
        const lines = buffer.split(/\r\n|\n/);
        buffer = lines.pop(); // Keep last partial line in buffer
        
        if (!headerRow && lines.length > 0) {
          headerRow = parser.parse(lines[0] + "\n").columns;
          lines.shift(); // Remove header
        }
        
        if (headerRow) {
          lines.forEach(line => {
            if (line.trim()) {
              const row = parser.parse(headerRow.join(",") + "\n" + line + "\n")[0];
              rowCallback(row);
            }
          });
        }
        
        return reader.read().then(processText);
      }
      
      return reader.read().then(processText);
    });
}

// Example usage of streaming
let count = 0;
streamCSV("large-data.csv", 
  row => {
    count++;
    if (count % 1000 === 0) {
      console.log(`Processed ${count} rows`);
    }
  },
  () => console.log(`Processing complete. Total: ${count} rows`)
);

// Local file handling (if in a browser environment)
// <input type="file" id="fileInput">
document.getElementById("fileInput").addEventListener("change", function(event) {
  const file = event.target.files[0];
  const reader = new FileReader();
  
  reader.onload = function(e) {
    const data = d3.csvParse(e.target.result);
    console.log(data);
  };
  
  reader.readAsText(file);
});
```all([
  d3.csv("data1.csv"),
  d3.json("data2.json")
]).then(([csv, json]) => {
  console.log(csv, json);
});
```

## Common Visualization Patterns

### Basic SVG Setup
```javascript
// Create SVG container with margins
const margin = {top: 20, right: 30, bottom: 40, left: 50};
const width = 800 - margin.left - margin.right;
const height = 500 - margin.top - margin.bottom;

// Append SVG element
const svg = d3.select("#chart")
  .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

// Add title for accessibility
svg.append("title")
  .text("Chart Title for Screen Readers");

// Add a background if needed
svg.append("rect")
  .attr("width", width)
  .attr("height", height)
  .attr("fill", "#f9f9f9");

// Create clipping path for bounded content
svg.append("defs")
  .append("clipPath")
    .attr("id", "clip")
  .append("rect")
    .attr("width", width)
    .attr("height", height);

// Create group with clipping
const chartArea = svg.append("g")
  .attr("clip-path", "url(#clip)");

// Add axis groups
const xAxisGroup = svg.append("g")
  .attr("class", "x-axis")
  .attr("transform", `translate(0,${height})`);
  
const yAxisGroup = svg.append("g")
  .attr("class", "y-axis");

// Add axes titles
svg.append("text")
  .attr("class", "x-axis-title")
  .attr("text-anchor", "middle")
  .attr("x", width / 2)
  .attr("y", height + margin.bottom - 5)
  .text("X Axis Title");

svg.append("text")
  .attr("class", "y-axis-title")
  .attr("text-anchor", "middle")
  .attr("transform", "rotate(-90)")
  .attr("x", -height / 2)
  .attr("y", -margin.left + 15)
  .text("Y Axis Title");

// Add legend group
const legend = svg.append("g")
  .attr("class", "legend")
  .attr("transform", `translate(${width - 100}, 20)`);

// Add grid lines
function makeXGridlines() {
  return d3.axisBottom(xScale).ticks(5);
}

function makeYGridlines() {
  return d3.axisLeft(yScale).ticks(5);
}

svg.append("g")
  .attr("class", "grid x-grid")
  .attr("transform", `translate(0,${height})`)
  .call(makeXGridlines()
    .tickSize(-height)
    .tickFormat("")
  );

svg.append("g")
  .attr("class", "grid y-grid")
  .call(makeYGridlines()
    .tickSize(-width)
    .tickFormat("")
  );

// Responsive SVG - resize with window
function responsivefy(svg) {
  // Get container and svg element
  const container = d3.select(svg.node().parentNode),
      width = parseInt(svg.style("width")),
      height = parseInt(svg.style("height")),
      aspect = width / height;

  // Add viewBox and preserve aspect ratio properties
  svg.attr("viewBox", `0 0 ${width} ${height}`)
     .attr("preserveAspectRatio", "xMidYMid meet")
     .call(resize);

  // Resize function
  function resize() {
    const targetWidth = parseInt(container.style("width"));
    svg.attr("width", targetWidth);
    svg.attr("height", Math.round(targetWidth / aspect));
  }

  // Add resize listener
  d3.select(window).on("resize." + container.attr("id"), resize);
}

// Apply responsiveness
svg.call(responsivefy);
```

### Bar Chart
```javascript
// Sample data
const data = [
  { name: "A", value: 10 },
  { name: "B", value: 20 },
  { name: "C", value: 30 },
  { name: "D", value: 25 },
  { name: "E", value: 15 }
];

// Create scales
const x = d3.scaleBand()
  .domain(data.map(d => d.name))           // X domain from names
  .range([0, width])
  .padding(0.1);                           // Padding between bars

const y = d3.scaleLinear()
  .domain([0, d3.max(data, d => d.value)]) // Y domain from max value
  .nice()                                  // Nice round values
  .range([height, 0]);                     // Range (inverted for SVG)

// Add horizontal gridlines
svg.append("g")
  .attr("class", "grid")
  .call(d3.axisLeft(y)
    .tickSize(-width)
    .tickFormat("")
  )
  .style("stroke-dasharray", "3,3")        // Dashed gridlines
  .style("stroke-opacity", 0.2);           // Faded gridlines

// Add bars
svg.selectAll(".bar")
  .data(data)
  .join("rect")                            // Join shorthand for enter-update-exit
    .attr("class", "bar")
    .attr("x", d => x(d.name))
    .attr("y", d => y(d.value))
    .attr("width", x.bandwidth())
    .attr("height", d => height - y(d.value))
    .attr("fill", "steelblue")
    .attr("rx", 2)                         // Rounded corners
    .attr("ry", 2);

// Add axes
svg.append("g")
  .attr("transform", `translate(0,${height})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .call(d3.axisLeft(y));

// Add labels
svg.selectAll(".label")
  .data(data)
  .join("text")
    .attr("class", "label")
    .attr("x", d => x(d.name) + x.bandwidth() / 2)
    .attr("y", d => y(d.value) - 5)
    .attr("text-anchor", "middle")
    .text(d => d.value);

// Animated bars
function updateBars(newData) {
  // Update scale domains
  x.domain(newData.map(d => d.name));
  y.domain([0, d3.max(newData, d => d.value)]).nice();
  
  // Update axes with transition
  svg.select(".x-axis")
    .transition()
    .duration(750)
    .call(d3.axisBottom(x));
    
  svg.select(".y-axis")
    .transition()
    .duration(750)
    .call(d3.axisLeft(y));
  
  // Update bars with transition
  svg.selectAll(".bar")
    .data(newData)
    .join(
      enter => enter.append("rect")
        .attr("class", "bar")
        .attr("x", d => x(d.name))
        .attr("y", height)                 // Start from bottom
        .attr("width", x.bandwidth())
        .attr("height", 0)                 // Start with height 0
        .attr("fill", "steelblue")
        .call(enter => enter.transition()
          .duration(750)
          .attr("y", d => y(d.value))
          .attr("height", d => height - y(d.value))
        ),
      update => update
        .transition()
        .duration(750)
        .attr("x", d => x(d.name))
        .attr("y", d => y(d.value))
        .attr("width", x.bandwidth())
        .attr("height", d => height - y(d.value)),
      exit => exit
        .transition()
        .duration(750)
        .attr("y", height)
        .attr("height", 0)
        .remove()
    );
}

// Horizontal bar chart variation
function createHorizontalBarChart(data) {
  // Swap x and y scales
  const x = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .nice()
    .range([0, width]);
    
  const y = d3.scaleBand()
    .domain(data.map(d => d.name))
    .range([0, height])
    .padding(0.1);
    
  // Add bars
  svg.selectAll(".bar")
    .data(data)
    .join("rect")
      .attr("class", "bar")
      .attr("x", 0)
      .attr("y", d => y(d.name))
      .attr("height", y.bandwidth())
      .attr("width", d => x(d.value))
      .attr("fill", "steelblue");
      
  // Add axes
  svg.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  svg.append("g")
    .call(d3.axisLeft(y));
}

// Grouped bar chart
function createGroupedBarChart(data) {
  // Data structure for grouped bars
  const groupedData = [
    { category: "A", values: { x: 5, y: 10, z: 15 } },
    { category: "B", values: { x: 8, y: 12, z: 20 } },
    { category: "C", values: { x: 15, y: 10, z: 5 } },
  ];
  
  const groups = groupedData.map(d => d.category);
  const subGroups = Object.keys(groupedData[0].values);
  
  // Scales
  const x = d3.scaleBand()
    .domain(groups)
    .range([0, width])
    .padding(0.2);
    
  const xSubgroup = d3.scaleBand()
    .domain(subGroups)
    .range([0, x.bandwidth()])
    .padding(0.05);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(groupedData, d => {
      return d3.max(Object.values(d.values));
    })])
    .nice()
    .range([height, 0]);
    
  const color = d3.scaleOrdinal()
    .domain(subGroups)
    .range(d3.schemeCategory10);
    
  // Add grouped bars
  svg.append("g")
    .selectAll("g")
    .data(groupedData)
    .join("g")
      .attr("transform", d => `translate(${x(d.category)},0)`)
      .selectAll("rect")
      .data(d => subGroups.map(key => ({key, value: d.values[key]})))
      .join("rect")
        .attr("x", d => xSubgroup(d.key))
        .attr("y", d => y(d.value))
        .attr("width", xSubgroup.bandwidth())
        .attr("height", d => height - y(d.value))
        .attr("fill", d => color(d.key));
        
  // Add axes
  svg.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  svg.append("g")
    .call(d3.axisLeft(y));
}

// Stacked bar chart
function createStackedBarChart(data) {
  // Data structure for stacked bars
  const stackedData = [
    { category: "A", values: { x: 5, y: 10, z: 15 } },
    { category: "B", values: { x: 8, y: 12, z: 20 } },
    { category: "C", values: { x: 15, y: 10, z: 5 } },
  ];
  
  const categories = stackedData.map(d => d.category);
  const subGroups = Object.keys(stackedData[0].values);
  
  // Stack generator
  const stack = d3.stack()
    .keys(subGroups)
    .value((d, key) => d.values[key]);
    
  const series = stack(stackedData);
  
  // Scales
  const x = d3.scaleBand()
    .domain(categories)
    .range([0, width])
    .padding(0.1);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(series, d => d3.max(d, d => d[1]))])
    .nice()
    .range([height, 0]);
    
  const color = d3.scaleOrdinal()
    .domain(subGroups)
    .range(d3.schemeCategory10);
    
  // Add stacked bars
  svg.append("g")
    .selectAll("g")
    .data(series)
    .join("g")
      .attr("fill", d => color(d.key))
      .selectAll("rect")
      .data(d => d)
      .join("rect")
        .attr("x", d => x(d.data.category))
        .attr("y", d => y(d[1]))
        .attr("width", x.bandwidth())
        .attr("height", d => y(d[0]) - y(d[1]));
        
  // Add axes
  svg.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  svg.append("g")
    .call(d3.axisLeft(y));
}
```

### Line Chart
```javascript
// Sample data (time series)
const data = [
  { date: new Date("2022-01-01"), value: 10 },
  { date: new Date("2022-02-01"), value: 20 },
  { date: new Date("2022-03-01"), value: 15 },
  { date: new Date("2022-04-01"), value: 25 },
  { date: new Date("2022-05-01"), value: 30 }
];

// Create scales
const x = d3.scaleTime()
  .domain(d3.extent(data, d => d.date))    // Min to max date
  .range([0, width]);

const y = d3.scaleLinear()
  .domain([0, d3.max(data, d => d.value)]) // 0 to max value
  .nice()                                  // Nice round values
  .range([height, 0]);

// Create line generator
const line = d3.line()
  .x(d => x(d.date))
  .y(d => y(d.value))
  .curve(d3.curveMonotoneX);               // Smooth curve

// Add grid lines
svg.append("g")
  .attr("class", "grid y-grid")
  .call(d3.axisLeft(y)
    .tickSize(-width)
    .tickFormat("")
  )
  .style("stroke-opacity", 0.1);

// Add line path
svg.append("path")
  .datum(data)
  .attr("class", "line")
  .attr("fill", "none")
  .attr("stroke", "steelblue")
  .attr("stroke-width", 2)
  .attr("d", line);

// Add data points
svg.selectAll(".dot")
  .data(data)
  .join("circle")
    .attr("class", "dot")
    .attr("cx", d => x(d.date))
    .attr("cy", d => y(d.value))
    .attr("r", 4)
    .attr("fill", "steelblue")
    .attr("stroke", "white")
    .attr("stroke-width", 1.5);

// Add axes
svg.append("g")
  .attr("class", "x-axis")
  .attr("transform", `translate(0,${height})`)
  .call(d3.axisBottom(x)
    .tickFormat(d3.timeFormat("%b %Y"))); // Format dates

svg.append("g")
  .attr("class", "y-axis")
  .call(d3.axisLeft(y));

// Add labels
svg.selectAll(".value-label")
  .data(data)
  .join("text")
    .attr("class", "value-label")
    .attr("x", d => x(d.date))
    .attr("y", d => y(d.value) - 10)
    .attr("text-anchor", "middle")
    .attr("font-size", "10px")
    .text(d => d.value);

// Animate line
function animateLine() {
  const path = svg.select(".line");
  const pathLength = path.node().getTotalLength();
  
  path
    .attr("stroke-dasharray", pathLength)
    .attr("stroke-dashoffset", pathLength)
    .transition()
    .duration(2000)
    .ease(d3.easeLinear)
    .attr("stroke-dashoffset", 0);
}

// Call animation
animateLine();

// Update line chart with new data
function updateLineChart(newData) {
  // Update scales
  x.domain(d3.extent(newData, d => d.date));
  y.domain([0, d3.max(newData, d => d.value)]).nice();
  
  // Update axes with transition
  svg.select(".x-axis")
    .transition()
    .duration(750)
    .call(d3.axisBottom(x)
      .tickFormat(d3.timeFormat("%b %Y")));
  
  svg.select(".y-axis")
    .transition()
    .duration(750)
    .call(d3.axisLeft(y));
  
  // Update line with transition
  svg.select(".line")
    .datum(newData)
    .transition()
    .duration(750)
    .attr("d", line);
  
  // Update dots with transition
  const dots = svg.selectAll(".dot")
    .data(newData);
  
  dots.exit()
    .transition()
    .duration(750)
    .attr("r", 0)
    .remove();
  
  dots.enter()
    .append("circle")
    .attr("class", "dot")
    .attr("r", 0)
    .attr("fill", "steelblue")
    .attr("stroke", "white")
    .attr("stroke-width", 1.5)
    .merge(dots)
    .transition()
    .duration(750)
    .attr("cx", d => x(d.date))
    .attr("cy", d => y(d.value))
    .attr("r", 4);
}

// Multiple lines
function createMultiLineChart(data) {
  // Data for multiple lines
  const multiData = [
    {
      name: "Series A",
      values: [
        { date: new Date("2022-01-01"), value: 10 },
        { date: new Date("2022-02-01"), value: 20 },
        { date: new Date("2022-03-01"), value: 15 },
        { date: new Date("2022-04-01"), value: 25 },
        { date: new Date("2022-05-01"), value: 30 }
      ]
    },
    {
      name: "Series B",
      values: [
        { date: new Date("2022-01-01"), value: 15 },
        { date: new Date("2022-02-01"), value: 10 },
        { date: new Date("2022-03-01"), value: 20 },
        { date: new Date("2022-04-01"), value: 15 },
        { date: new Date("2022-05-01"), value: 25 }
      ]
    }
  ];
  
  // Get all dates and values for domains
  const allDates = multiData.flatMap(d => d.values.map(v => v.date));
  const allValues = multiData.flatMap(d => d.values.map(v => v.value));
  
  // Create scales
  const x = d3.scaleTime()
    .domain(d3.extent(allDates))
    .range([0, width]);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(allValues)])
    .nice()
    .range([height, 0]);
    
  // Color scale
  const color = d3.scaleOrdinal(d3.schemeCategory10)
    .domain(multiData.map(d => d.name));
    
  // Line generator
  const line = d3.line()
    .x(d => x(d.date))
    .y(d => y(d.value))
    .curve(d3.curveMonotoneX);
    
  // Add multiple lines
  svg.selectAll(".line")
    .data(multiData)
    .join("path")
      .attr("class", "line")
      .attr("fill", "none")
      .attr("stroke", d => color(d.name))
      .attr("stroke-width", 2)
      .attr("d", d => line(d.values));
      
  // Add axes
  svg.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  svg.append("g")
    .call(d3.axisLeft(y));
    
  // Add legend
  const legend = svg.append("g")
    .attr("class", "legend")
    .attr("transform", `translate(${width - 100}, 20)`);
    
  legend.selectAll(".legend-item")
    .data(multiData)
    .join("g")
      .attr("class", "legend-item")
      .attr("transform", (d, i) => `translate(0, ${i * 20})`)
      .each(function(d) {
        const g = d3.select(this);
        
        g.append("line")
          .attr("x1", 0)
          .attr("x2", 20)
          .attr("stroke", color(d.name))
          .attr("stroke-width", 2);
          
        g.append("text")
          .attr("x", 25)
          .attr("y", 5)
          .text(d.name)
          .style("font-size", "12px");
      });
}

// Area chart variation
function createAreaChart(data) {
  // Area generator
  const area = d3.area()
    .x(d => x(d.date))
    .y0(height)                            // Baseline
    .y1(d => y(d.value))                   // Top line
    .curve(d3.curveMonotoneX);             // Smooth curve
    
  // Add area path
  svg.append("path")
    .datum(data)
    .attr("class", "area")
    .attr("fill", "steelblue")
    .attr("fill-opacity", 0.3)
    .attr("stroke", "none")
    .attr("d", area);
    
  // Add line on top of area
  svg.append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 2)
    .attr("d", line);
}

// Stacked area chart
function createStackedAreaChart(data) {
  // Data for stacked areas
  const stackedData = [
    { date: new Date("2022-01-01"), a: 10, b: 15, c: 5 },
    { date: new Date("2022-02-01"), a: 20, b: 10, c: 8 },
    { date: new Date("2022-03-01"), a: 15, b: 20, c: 12 },
    { date: new Date("2022-04-01"), a: 25, b: 15, c: 10 },
    { date: new Date("2022-05-01"), a: 30, b: 25, c: 15 }
  ];
  
  const keys = ["a", "b", "c"];
  
  // Stack generator
  const stack = d3.stack()
    .keys(keys)
    .order(d3.stackOrderNone)
    .offset(d3.stackOffsetNone);
    
  const series = stack(stackedData);
  
  // Colors
  const color = d3.scaleOrdinal()
    .domain(keys)
    .range(d3.schemeCategory10);
    
  // Scales
  const x = d3.scaleTime()
    .domain(d3.extent(stackedData, d => d.date))
    .range([0, width]);
    
  const y = d3.scaleLinear()
    .domain([0, d3.max(series, d => d3.max(d, d => d[1]))])
    .nice()
    .range([height, 0]);
    
  // Area generator
  const area = d3.area()
    .x(d => x(d.data.date))
    .y0(d => y(d[0]))
    .y1(d => y(d[1]))
    .curve(d3.curveMonotoneX);
    
  // Add stacked areas
  svg.selectAll(".area")
    .data(series)
    .join("path")
      .attr("class", "area")
      .attr("fill", d => color(d.key))
      .attr("fill-opacity", 0.7)
      .attr("d", area);
      
  // Add axes
  svg.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));
    
  svg.append("g")
    .call(d3.axisLeft(y));
}
```

### Scatter Plot
```javascript
// Create scales
const x = d3.scaleLinear()
  .domain(d3.extent(data, d => d.x))
  .range([0, width]);

const y = d3.scaleLinear()
  .domain(d3.extent(data, d => d.y))
  .range([height, 0]);

// Add circles
svg.selectAll("circle")
  .data(data)
  .join("circle")
    .attr("cx", d => x(d.x))
    .attr("cy", d => y(d.y))
    .attr("r", 5)
    .attr("fill", "steelblue");

// Add axes
svg.append("g")
  .attr("transform", `translate(0,${height})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .call(d3.axisLeft(y));
```

## D3.js Version Differences

### D3 v7 (Latest)
```javascript
// Fetch API instead of Xhr
d3.json("data.json").then(data => {...});

// Native ES modules
import {select, scaleLinear} from "d3";

// Modern Web API
selection.on("mouseenter", (event, d) => {...});  // event is separate
```

### D3 v6
```javascript
// Event handling with separate event object
selection.on("click", (event, d) => {...});
```

### D3 v5
```javascript
// Promises instead of callbacks
d3.json("data.json").then(data => {...});

// Collection.join() for enter/update/exit
selection.join("circle");
```

### D3 v4
```javascript
// Modular structure - no more d3.scale, use d3.scaleLinear etc.
const x = d3.scaleLinear();  // Not d3.scale.linear()

// Named exports
import {select} from "d3-selection";
```