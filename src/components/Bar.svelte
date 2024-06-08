<script>
   import { scaleLinear } from 'd3-scale';

   // From Data Cleaning
   const categories = [
        {category: "Bars", count: 427},
		{category: "American", count: 359},
		{category: "Mexican", count: 192},
        {category: "Sandwiches", count: 173},
        {category: "Breakfast", count: 170},
        {category: "Brunch", count: 170},
        {category: "Tea", count: 131},
        {category: "Coffee", count: 120},
        {category: "Pizza", count: 113},
        {category: "Seafood", count: 101},
        {category: "Burgers", count: 100},
        {category: "Italian", count: 83}
	];

    const xTicks = ["Bars", "American", "Mexican", "Sandwiches", "Breakfast", "Brunch", "Tea", "Coffee", "Pizza", "Wine", "Seafood", "Burgers", "Italian"];
	const yTicks = [0, 100, 200, 300, 400, 500];
	const padding = {top: 20, right: 20, bottom: 20, left: 20};

    let width = 800;
    let height = 400;

    function formatMobile(tick) {
		return "'" + tick.toString().slice(-2);
	}

    $: xScale = scaleLinear()
		.domain([0, xTicks.length-1.4])
		.range([padding.left, width - padding.right]);

	$: yScale = scaleLinear()
		.domain([0, Math.max.apply(null, yTicks)])
		.range([height - padding.bottom, padding.top]);

	$: innerWidth = width - (padding.left + padding.right);
	$: barWidth = innerWidth / xTicks.length;

    let hovered = -1;
    let recorded_mouse_position = {
        x: 0, y: 0
    };

</script>
  
<main>
    <div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
        <svg>
            <!-- y axis -->
            <g class="axis y-axis">
                {#each yTicks as tick}
                    <g class="tick tick-{tick}" transform="translate(0, {yScale(tick)})">
                        <line x2="100%" />
                        <text y="-4">{tick} {tick === 500 ? ' Restaurants' : ''}</text>
                    </g>
                {/each}
            </g>
    
            <!-- x axis -->
            <g class="axis x-axis">
                {#each categories as category, i}
                    <g class="tick" transform="translate({xScale(i)},{height})">
                        <text x={barWidth / 1.8} y="-9">{width > 380 ? category.category : formatMobile(category.category)}</text>
                    </g>
                {/each}
            </g>
    
            <g class="bars">
                {#each categories as category, i}
                    <rect
                        x={xScale(i) + 6}
                        y={yScale(category.count)}
                        width={barWidth - 6}
                        height={yScale(0) - yScale(category.count)}
                    
                        fill={i === hovered ? "#1B45A6": "#6EC0E9"}
                        stroke={i === hovered ? "black": "white"}
                        stroke-width={i === hovered ? "2px": "0px"}
				        on:mouseover={(event) => {
                            hovered = i;
                            recorded_mouse_position = {
                                x: event.pageX,
                                y: event.pageY
                            };
                            }}
                        on:mouseout={(event) => { hovered = -1; }}
                        style="opacity: 0.65;"
                        />
				        
                    
                {/each}
            </g>
        </svg>
    </div>
    <div
		class={hovered === -1 ? "tooltip-hidden": "tooltip-visible"}
        style="left: {recorded_mouse_position.x - 280}px; top: 
        {recorded_mouse_position.y - 100}px"	
	>
		{#if hovered !== -1}
			Out of 1159 restaurants in Santa Barbara,
            there are <strong>{categories[hovered].count}</strong> restaurants 
            in the <strong>{categories[hovered].category}</strong> category.
		{/if}
	</div>

</main>
  
<style>
    h2 {
		text-align: center;
        font-size: 30px;
        font-family: American Typewriter, serif;
	}

	.chart {
		width: 100%;
		max-width: 800px;
		margin: 0 auto;
	}

	svg {
		position: relative;
		width: 100%;
		height: 400px;
	}

	.tick {
		font-family: American Typewriter, serif;
		font-size: 0.350em;
		font-weight: 200;
	}

	.tick line {
		stroke: #e2e2e2;
		stroke-dasharray: 2;
	}

	.tick text {
		fill: #444444;
		text-anchor: start;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.x-axis .tick text {
		text-anchor: middle;
	}
    
    .tooltip-hidden {
		visibility: hidden;
		font-family: American Typewriter, serif;
		width: 200px;
		position: absolute;
        opacity: 0;
	}

	.tooltip-visible {
		font: 15px Arial;
		font-family: American Typewriter, serif;
		visibility: visible;
		background-color: #ADD8E6;
		border-radius: 10px;
		width: 200px;
		color: black;
		position: absolute;
		padding: 10px;
        border: 1px solid rgba(0, 0, 0, 0.5);
        pointer-events: none;
	}
</style>