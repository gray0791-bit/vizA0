<script>
    let maxClick = $state(5);
    let cnt = $state(0); // tip: https://svelte.dev/docs/svelte/$state
    
  
    function onClick() {
      // tip: Since DOM (i.e., the webpage content) will automatically update based on values, 
      // [<p id="info">Remaining Number of Clicks: {cnt}</p>]
      // we only need to change the cnt number here. 
      cnt -= 1
    }
  </script>
  
  <h1>Genevieve's VIS Site</h1>
  <img
    width="200px"
    src="https://images.squarespace-cdn.com/content/v1/644442852d7ed23ae36bba07/1682195096669-6230QVCD1DA03K6Z1PR3/PXL_20230325_192928191.MP.jpg?format=2500w"
  />
  <div>
    You can click up to
    <select 
        bind:value={maxClick} 
        onchange={() => (cnt = maxClick)}>
      {#each [2, 4, 6] as optionNum}
        <option value={optionNum}>
          {optionNum}
        </option>
      {/each}
    </select>
    times
  </div>
  <button onclick={onClick}> Click Me </button>

  <!-- tip: use {#if...} template syntax here (https://svelte.dev/docs/svelte/if) so that the content below will automatically update when cnt value changes -->
    <!-- `the content below should only show when cnt >0` -->
     {#if cnt > 0}
        <p id="info">Remaining Number of Clicks: {cnt}</p>
     {:else}
        <!-- The content below should only shown when cnt =0 -->
        <p>No more clicks allowed</p>
     {/if}
  
  
  <style>
    body {
      font-family: Arial, Helvetica, sans-serif;
    }
    button {
      background-color: #44aa66;
      /* background-color: blue; */
      color: white;
      font-size: xx-large;
      padding: 10px 20px;
      border: none;
      cursor: pointer;
      border-radius: 5px;
    }
  </style>
  