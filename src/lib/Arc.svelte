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
  let { movies, progress = 100, width = 900, height = 500 }: Props = $props();

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

  const links = $derived((() => { 
    let connections = [];
    let counts: { [genre: string]: number } = {};
    movies
      .filter(m => m.year <= upYear) // Keep it consistent with your other charts!
      .forEach(movie => {
        for (let i = 0; i < movie.genres.length; i++) {
            for (let j = i + 1; j < movie.genres.length; j++) {
                const g1 = movie.genres[i];
                const g2 = movie.genres[j];
                const pairKey = [g1, g2].sort().join("-"); 

                counts[pairKey] = (counts[pairKey] || 0) + 1;
            }
        }
      });

      Object.entries(counts).forEach(([pairString, tally]) => {
  
        const [genre1, genre2] = pairString.split("-");
        connections.push({
            source: genre1,
            target: genre2,
            count: tally
        });
      });
    return connections;
  })());

  const sortedGenres = $derived([...genres].sort());

  const xScale = $derived(
    d3.scalePoint()
      .domain(sortedGenres)
      .range([usableArea.left, usableArea.right])
  );

</script>

<h3>
  Q2: What are the correlations between different genres? For example, which genre often co-occurs with comedy in a movie?
  <p>
    Romance and Comedy have a large overlap. Comedy and Family also overlap a good bit.
  </p>
</h3>


{#if movies.length > 0}
  <svg {width} {height}>
     <g class="links">
    {#each links as link}
      {@const xStart = xScale(link.source)}
      {@const xEnd = xScale(link.target)}
      {@const radius = Math.abs(xEnd - xStart) / 2}
    
      <path
        d="M {xStart} {usableArea.bottom} A {radius} {radius} 0 0 1 {xEnd} {usableArea.bottom}"
        fill="none"
        stroke="green"
        stroke-opacity="0.2"
        stroke-width={link.count}
      />
    {/each}
  </g>
    {#each sortedGenres as genre}
      <circle 
        cx={xScale(genre)} 
        cy={usableArea.bottom} 
        r="4" 
        fill="grey" 
      />
      <text 
        x={xScale(genre)} 
        y={usableArea.bottom + 20} 
        text-anchor="middle" 
        font-size="10"
        transform="rotate(45, {xScale(genre)}, {usableArea.bottom + 20})"
      >
        {genre}
      </text>
    {/each}
  </svg>
  
{/if}