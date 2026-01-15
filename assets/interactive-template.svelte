<script>
  import * as d3 from 'd3';

  let { data = [] } = $props();

  let svgElement = $state();
  let tooltipElement = $state();
  let selectedPoint = $state(null);

  $effect(() => {
    if (!data || data.length === 0 || !svgElement) return;

    const svg = d3.select(svgElement);
    svg.selectAll("*").remove();

    // Dimensions
    const width = 800;
    const height = 500;
    const margin = { top: 20, right: 30, bottom: 40, left: 50 };
    const innerWidth = width - margin.left - margin.right;
    const innerHeight = height - margin.top - margin.bottom;

    // Create main group
    const g = svg.append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    // Scales
    const xScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.x)])
      .range([0, innerWidth])
      .nice();

    const yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.y)])
      .range([innerHeight, 0])
      .nice();

    const sizeScale = d3.scaleSqrt()
      .domain([0, d3.max(data, d => d.size || 10)])
      .range([3, 20]);

    const colourScale = d3.scaleOrdinal(d3.schemeCategory10);

    // Add zoom behaviour
    const zoom = d3.zoom()
      .scaleExtent([0.5, 10])
      .on("zoom", (event) => {
        g.attr("transform", `translate(${margin.left + event.transform.x},${margin.top + event.transform.y}) scale(${event.transform.k})`);
      });

    svg.call(zoom);

    // Axes
    const xAxis = d3.axisBottom(xScale);
    const yAxis = d3.axisLeft(yScale);

    g.append("g")
      .attr("class", "x-axis")
      .attr("transform", `translate(0,${innerHeight})`)
      .call(xAxis);

    g.append("g")
      .attr("class", "y-axis")
      .call(yAxis);

    // Grid lines
    g.append("g")
      .attr("class", "grid")
      .attr("opacity", 0.1)
      .call(d3.axisLeft(yScale)
        .tickSize(-innerWidth)
        .tickFormat(""));

    g.append("g")
      .attr("class", "grid")
      .attr("opacity", 0.1)
      .attr("transform", `translate(0,${innerHeight})`)
      .call(d3.axisBottom(xScale)
        .tickSize(-innerHeight)
        .tickFormat(""));

    // Tooltip
    const tooltip = d3.select(tooltipElement);

    // Data points
    const circles = g.selectAll("circle")
      .data(data)
      .join("circle")
      .attr("cx", d => xScale(d.x))
      .attr("cy", d => yScale(d.y))
      .attr("r", d => sizeScale(d.size || 10))
      .attr("fill", d => colourScale(d.category || 'default'))
      .attr("stroke", "#fff")
      .attr("stroke-width", 2)
      .attr("opacity", 0.7)
      .style("cursor", "pointer");

    // Hover interactions
    circles
      .on("mouseover", function(event, d) {
        // Enlarge circle
        d3.select(this)
          .transition()
          .duration(200)
          .attr("opacity", 1)
          .attr("stroke-width", 3);

        // Show tooltip
        tooltip
          .style("display", "block")
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 10) + "px")
          .html(`
            <strong>${d.label || 'Point'}</strong><br/>
            X: ${d.x.toFixed(2)}<br/>
            Y: ${d.y.toFixed(2)}<br/>
            ${d.category ? `Category: ${d.category}<br/>` : ''}
            ${d.size ? `Size: ${d.size.toFixed(2)}` : ''}
          `);
      })
      .on("mousemove", function(event) {
        tooltip
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 10) + "px");
      })
      .on("mouseout", function() {
        // Restore circle
        d3.select(this)
          .transition()
          .duration(200)
          .attr("opacity", 0.7)
          .attr("stroke-width", 2);

        // Hide tooltip
        tooltip.style("display", "none");
      })
      .on("click", function(event, d) {
        // Highlight selected point
        circles.attr("stroke", "#fff").attr("stroke-width", 2);
        d3.select(this)
          .attr("stroke", "#000")
          .attr("stroke-width", 3);

        selectedPoint = d;
      });

    // Add transition on initial render
    circles
      .attr("r", 0)
      .transition()
      .duration(800)
      .delay((d, i) => i * 20)
      .attr("r", d => sizeScale(d.size || 10));

    // Axis labels
    g.append("text")
      .attr("class", "axis-label")
      .attr("x", innerWidth / 2)
      .attr("y", innerHeight + margin.bottom - 5)
      .attr("text-anchor", "middle")
      .style("font-size", "14px")
      .text("X Axis");

    g.append("text")
      .attr("class", "axis-label")
      .attr("transform", "rotate(-90)")
      .attr("x", -innerHeight / 2)
      .attr("y", -margin.left + 15)
      .attr("text-anchor", "middle")
      .style("font-size", "14px")
      .text("Y Axis");
  });
</script>

<div class="relative">
  <svg
    bind:this={svgElement}
    width="800"
    height="500"
    style="border: 1px solid #ddd; cursor: grab;"
  ></svg>
  <div
    bind:this={tooltipElement}
    style="
      position: absolute;
      display: none;
      padding: 10px;
      background: white;
      border: 1px solid #ddd;
      border-radius: 4px;
      pointer-events: none;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      font-size: 13px;
      z-index: 1000;
    "
  ></div>
  {#if selectedPoint}
    <div class="mt-4 p-4 bg-blue-50 rounded border border-blue-200">
      <h3 class="font-bold mb-2">Selected Point</h3>
      <pre class="text-sm">{JSON.stringify(selectedPoint, null, 2)}</pre>
    </div>
  {/if}
</div>
