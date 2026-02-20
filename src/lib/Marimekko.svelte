<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";
  // define the props of the Bar component
  type Props = {
    movies: TMovie[];
    progress?: number;
    width?: number;
    height?: number;
  };
  // progress is 100 by default unless specified
  let { movies, progress = 100, width = 1300, height = 500 }: Props = $props();

  let selectedGenre: string = $state(''); //give starting value 

  // processing the data; $derived is used to create a reactive variable that updates whenever the dependent variables change
  const yearRange = $derived(d3.extent(movies.map((d) => d.year)));

  function getUpYear(yearRange: [undefined, undefined] | [Date, Date]) {
    if (!yearRange[0]) return new Date();
    const timeScale = d3.scaleTime().domain(yearRange).range([0, 100]);
    return timeScale.invert(progress);
  }
  const upYear: Date = $derived(getUpYear(yearRange!));

  function getGenreNums(movies: TMovie[], upYear: Date) {
    let res: { [genre: string]: number } = {};
    movies
      .filter((movie) => movie.year <= upYear)
      .forEach((movie) => {
        movie.genres.forEach((genre: string) => {
          res[genre] = (res[genre] || 0) + 1;
        });
      });
    return res;
  }

  const genreNums = $derived(getGenreNums(movies, upYear));

  const margin = {
    top: 15,
    bottom: 50,
    left: 100,
    right: 10,
  };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };

// processing the data; $derived is used to create a reactive variable that updates whenever the dependent variables change
  //const yearRange = $derived(d3.extent(movies.map((d) => d.year)));

const genres = $derived([...new Set(movies.flatMap(m => m.genres))]);

const expandedData = $derived(
  movies.flatMap(movie => 
    movie.genres.map(genre => ({ 
      year: movie.year.getFullYear(), 
      genre: genre 
    }))
  )
);

const yearlyGenreCounts = $derived(
  d3.rollups(
    expandedData,
    (v) => v.length, // The "reducer": count how many items are in the group
    (d) => d.year,   // First group by: Year
    (d) => d.genre   // Second group by: Genre
  )
);

$effect(() => {
  console.log("Rolled up data:", yearlyGenreCounts);
});

const marimekkoData = $derived((() => {
  let rects = [];
  let currentX = 0; // Tracks the cumulative width along the X axis

  
  
  // Sort the years chronologically so the X-axis reads left-to-right
  const sortedYears = [...yearlyGenreCounts].sort((a, b) => a[0] - b[0]);

  sortedYears.forEach(([year, genresMap]) => {
    // Total movies for this specific year
    const yearTotal = d3.sum(genresMap, d => d[1]); 
    let currentY = 0; // Starts at the bottom!
    
    // NEW: Sort the genres for this specific year from largest to smallest count
    const sortedGenresForYear = [...genresMap].sort((a, b) => b[1] - a[1]);

    // Loop through our newly sorted list instead of the raw map
    sortedGenresForYear.forEach(([genre, count]) => {
      const heightPct = count / yearTotal; 
      
      rects.push({
        year,
        genre,
        count,
        x0: currentX,                  
        x1: currentX + yearTotal,      
        y0: currentY,                  
        y1: currentY + heightPct       
      });
      
      currentY += heightPct; 
    });
    
    currentX += yearTotal; 
  });
  
  return { rects, totalCount: currentX };
})());

const yearColumns = $derived((() => {
  let cols = [];
  let currentX = 0;
  
  const sortedYears = [...yearlyGenreCounts].sort((a, b) => a[0] - b[0]);
  
  sortedYears.forEach(([year, genresMap]) => {
    const yearTotal = d3.sum(genresMap, d => d[1]);
    cols.push({ 
      year: year, 
      center: currentX + (yearTotal / 2) // Find the exact middle of the column
    });
    currentX += yearTotal;
  });
  
  return cols;
})());

// 2. The Scales
const xLinearScale = $derived(
  d3.scaleLinear()
    .domain([0, marimekkoData.totalCount])
    .range([usableArea.left, usableArea.right])
);

const yLinearScale = $derived(
  d3.scaleLinear()
    .domain([0, 1]) // 0 to 1 (100%)
    .range([usableArea.bottom, usableArea.top]) // SVG draws top-down, so we invert this!
);

const sortedGenres = $derived([...genres].sort());

// 3. A Color Scale to tell the genres apart
const colorScale = $derived(
  d3.scaleOrdinal()
    .domain(sortedGenres)
    .range(d3.quantize(t => d3.interpolateRainbow(t), sortedGenres.length))
);

  const xScale = $derived(
    d3.scalePoint()
      .domain(sortedGenres)
      .range([usableArea.left, usableArea.right])
  );

</script>

<h3>
  Q1: How do the annual top 3 genres change over time? For example, which genre is always within the top3?
  <p>
    Comedy and drama are almost always the top 2 with romance regularly poping into the top 3, in the mid 2010s it came in 2nd often.
  </p>
</h3>


{#if movies.length > 0}
  <svg {width} {height}>
    <g class="marimekko-blocks">
      {#each marimekkoData.rects as d}
      <rect
        x={xLinearScale(d.x0)}
        y={yLinearScale(d.y1)} 
        width={xLinearScale(d.x1) - xLinearScale(d.x0)}
        height={yLinearScale(d.y0) - yLinearScale(d.y1)}
        fill={colorScale(d.genre)}
        stroke="white"
        stroke-width="0.5"
      >
        <title>{d.year} - {d.genre}: {d.count} movies</title>
      </rect>
      {/each}
    </g>
    <g class="x-axis-labels">
    {#each yearColumns as col}
      <text
        x={xLinearScale(col.center)}
        y={usableArea.bottom + 20}
        font-size="10"
        text-anchor="middle"
        transform="rotate(45, {xLinearScale(col.center)}, {usableArea.bottom + 20})"
      >
        {col.year}
      </text>
    {/each}
  </g>

  <g class="legend" transform="translate(10, {usableArea.top})">
    {#each sortedGenres as genre, i}
      <rect 
        x="0" 
        y={i * 15} 
        width="10" 
        height="10" 
        fill={colorScale(genre)} 
      />
      <text 
        x="15" 
        y={i * 15 + 9} 
        font-size="10" 
        text-anchor="start"
      >
        {genre}
      </text>
    {/each}
  </g>
  </svg>
  
{/if}