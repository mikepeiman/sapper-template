<script>
  import { onMount } from "svelte";
  import { writable } from "svelte/store"
  import { galaxy } from "../store"
  import seedrandom from "seedrandom";
  // import '../components/galaxy-generator.js'

//  galaxy = galaxy.useLocalStorage()
  $: galaxy = [];
  $: galSize = 15;
  $: warpMin = 3;
  $: warpMax = 3;
  // let galSize = 5;


  onMount(async () => {
     warpMin = document.getElementById('warpsMin').value
     warpMax = document.getElementById('warpsMax').value
     galSize = document.getElementById('galSize').value

    generateGalaxy(galSize, warpMin, warpMax).then(g => {
      linkGalaxy(g)
      localStorage.setItem("galaxy", JSON.stringify(g));
      initGalaxy()
    })
  });

  function newGalaxy() {
    warpMin = document.getElementById('warpsMin').value
    warpMax = document.getElementById('warpsMax').value
    galSize = document.getElementById('galSize').value

      generateGalaxy(galSize, warpMin, warpMax).then(g => {
      // localStorage.setItem("galaxy", JSON.stringify(g))
      linkGalaxy(g)
      localStorage.setItem("galaxy", JSON.stringify(g));
      // return galaxy = g
      initGalaxy()
    })
  }
  
  async function initGalaxy() {
        galaxy = JSON.parse(localStorage.getItem("galaxy"));
    console.log(galaxy)
    return galaxy = galaxy
  }

  function loadGalaxy() {
    console.log('loadGalaxy clicked')
    // galaxy = [{id: 0, name: "test"},{id: 1, name: "Mike"}]
    galaxy = JSON.parse(localStorage.getItem("galaxy"));
    console.log(galaxy)
    return galaxy = galaxy
  }

  async function generateGalaxy(galSize, warpMin, warpMax) {
    // more possible settings:
    // warpDensity, oneWayWarpDensity
    let galaxy = []
    for (let syscount = 0; syscount < galSize; syscount++) {
      galaxy[syscount] = await makeSector(syscount, warpMin, warpMax);
    }
    // localStorage.setItem("galaxy", JSON.stringify(galaxy));
    return galaxy
  }

  
  function makeSector(syscount, warpMin, warpMax) {
    let sector = {};
    sector.id = syscount;
    sector.warpsQuota = getRandomInt(warpMin, warpMax);
    sector.inlinks = [];
    sector.outlinks = [];
    sector.accepting = {
      inlinks: true,
      outlinks: true
    }
    return sector
  }


  function linkGalaxy(galaxy) {
    // let galaxy = JSON.parse(localStorage.getItem("galaxy"));
    console.log(`from linkGalaxy(), localStorage galaxy: `)
    console.log(galaxy)
    galaxy.forEach(sector => {
      setOutlinks(sector, galaxy)
    });
    // localStorage.setItem("galaxy", JSON.stringify(galaxy));
    //     console.log(`from linkGalaxy(), localStorage galaxy: `)
    // console.log(galaxy)
    return galaxy
  }

  async function setOutlinks(sector, galaxy) {
    let needsLinks = sector.warpsQuota - sector.outlinks
    let warps = []
    getValidSectorToLinkTo(needsLinks, 0, warps, sector, galaxy)
  }

  async function getValidSectorToLinkTo(count, overflow, warps, sector, galaxy) {
    // console.log(`@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@`) 
    // console.log(`sector.id ${sector.id}  |||  count ${count}  |||  sector.warpsQuota ${sector.warpsQuota}`)
    // console.log(`@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@`) 
    // console.log(`getValidSectorToLinkTo head: warps array currently empty: ${warps}`)
    // console.log(`getValidSectorToLinkTo head: overflow counter currently: ${overflow}`)
    // console.log('\n\n')

    while(count > 0) {
      // get random sector
      let rand = getRandomSector(galaxy)
      
      // check if it is valid:
      //    does not match current sector
      //    is not already in outlinks
      //    target has not hit warpsQuota >= inlinks.length or outlinks.length

      // console.log(`rand.id ${rand.id} !== sector.id ${sector.id} --- ${rand.id !== sector.id}`)
      // console.log(`!sector.outlinks[ ${sector.outlinks} ].includes(rand.id) ${rand.id} --- ${!sector.outlinks.includes(rand.id)}`)
      // console.log(`rand.outlinks.length ${rand.outlinks.length} <= rand.warpsQuota ${rand.warpsQuota} --- ${rand.outlinks.length <= rand.warpsQuota}`)

      if(rand.id !== sector.id
        && !sector.outlinks.includes(rand.id)
        && rand.outlinks.length <= rand.warpsQuota) 
      // if(rand.id !== sector.id) {
      //   if(!sector.outlinks.includes(rand.id)) {
      //     if(rand.outlinks.length <= rand.warpsQuota) {
      //       console.log(`@@@@@@@@@@@@@@@@@       sector [[[ ${sector.id} ]]] warp ### ${rand.id} ###  passed all tests  @@@@@@@@@@@@@@@@@@@@`)
      //       warps.push(rand.id)
      //       warps.sort()
      //       console.log(`Warps array currently: ${warps}`)
      //       sector.outlinks = warps
      //       sector.inlinks = warps
      //       count--
      //       overflow = 0
      //     }
      //   }
      // }
      {
        console.log(`@@@@@@@@@@@@@@@@@       sector [[[ ${sector.id} ]]] warp ### ${rand.id} ###  passed all tests  @@@@@@@@@@@@@@@@@@@@`)
        if(rand.id !== sector.id) {console.log(`rand.id ${rand.id} !== sector.id ${sector.id} --- ${rand.id !== sector.id}`)}
        if(!sector.outlinks.includes(rand.id)) {console.log(`!sector.outlinks[ ${sector.outlinks} ].includes(rand.id) ${rand.id} --- ${!sector.outlinks.includes(rand.id)}`)}
        if(rand.outlinks.length <= rand.warpsQuota) {console.log(`rand.outlinks.length ${rand.outlinks.length} <= rand.warpsQuota ${rand.warpsQuota} --- ${rand.outlinks.length <= rand.warpsQuota}`)}
        warps.push(rand.id)
        warps.sort()
        console.log(`Warps array currently: ${warps}`)
        sector.outlinks = warps
        sector.inlinks = warps
        count--
        overflow = 0
      } else {
        ++overflow
        console.log('\n\n')
        console.log(`##########  We seem to have a problem >>> WARP ${rand.id} <<<  OVERFLOW ### ${overflow} ### Not passing tests #############`)
        if(rand.id === sector.id) 
          {
            console.log(`rand.id ${rand.id} === sector.id ${sector.id} --- ${rand.id === sector.id}`)
            }
        if(sector.outlinks.includes(rand.id)) 
          {
            console.log(`sector.outlinks[ ${sector.outlinks} ].includes(rand.id) ${rand.id} --- ${sector.outlinks.includes(rand.id)}`)
            }
        if(rand.outlinks.length > rand.warpsQuota) 
          {
            console.log(`rand.outlinks.length ${rand.outlinks.length} > rand.warpsQuota ${rand.warpsQuota} --- ${rand.outlinks.length > rand.warpsQuota}`)
            rand.outlinks = []
            }
        console.log('\n\n')
        
        if(overflow > 10) {
          console.log('Hit overflow of over 10 failed link tests! Galaxy size must be too small to accomodate warp conditions.')
          // return "Overflow error"
          count = 0
          break
        } else {
          getValidSectorToLinkTo(count, overflow, warps, sector, galaxy)
        }
      }
      if(overflow > 10) {
          console.log('Hit overflow of over 10 failed link tests! Galaxy size must be too small to accomodate unique warps.')
          // return "Overflow error"
          count = 0
          break
        }
    }
    // sector.outlinks = warps
    // sector.inlinks = warps
    // return warps
  }

  function getRandomSector(galaxy) {
    return galaxy[Math.floor(Math.random() * galaxy.length)];
  }

  function getRandomInt(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    let prng = seedrandom();
    let randomInRange = Math.floor(prng() * (max - min + 1)) + min;
    // console.log(`getRandomInt prng: ${prng()}, in range ${randomInRange}`)
    return randomInRange;
  }

  function getSector(id) {
    return galaxy[id];
  }

  class Sector {
    constructor(id, name, numOutlinks, outlinks, inlinks) {
      this.id = id;
      this.name = name;
      this.numOutlinks = numOutlinks;
      this.outlinks = outlinks;
      this.inlinks = inlinks;
    }

    getWarps() {
      return `Warps from this sector are ${this.warps}`;
    }
  }

  class Ship {
    constructor(id, name, baseCost, holds, moves) {
      this.id = id;
      this.name = name;
      this.baseCost = baseCost;
      this.holds = holds;
      this.moves = moves;
    }

    getCost() {
      return `Today's price is $${baseCost * 1.5}}`;
    }
  }
</script>

<style lang="scss">
  .tw2002 {
    background: rgba(0, 0, 0, 0.15);
    padding: 1rem;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }
  .game-menu {
    display: flex;
    flex-direction: row;
  }

  button {
    background: rgba(0, 0, 255, 0.25);
    outline: none;
    border: 1px solid rgba(155, 25, 255, 1);
    padding: 0.5rem;
    margin: 0 1rem 1rem 0;
    box-shadow: none;
    transition: all 0.25s;
    &:hover {
      background: rgba(255, 255, 255, 0.25);
      border-radius: 5px;
    }
  }
  #start-game {
  }

  .sector-list {
    width: 100vw;
    height: auto;
    padding: 1rem;
    background: rgba(55, 155, 175, 0.35);
  }

  .svelte-universe {
    display: grid;
    grid-template-columns: 20% 1fr;
    grid-template-rows: 2rem auto;
  }

  .sector-warps {
    display: grid;
    grid-template-columns: 100px 75px 75px repeat(10, 5.25ch);
    grid-gap: 1ch;
  }
  .warp {
    padding: 0.25rem;
    margin-right: 0.25rem;
    width: 5ch;
    background: rgba(0, 0, 0, 0.1);
    border-bottom: 3px solid rgba(0, 50, 250, .5);
    transition: all .15s;
    &:hover {
      background: rgba(255,155,205, 0.25);
      border-bottom: 3px solid rgba(0, 50, 250, .75);
    }
  }
</style>

<svelte:head>
  <title>About</title>
</svelte:head>
<section class="tw2002">
  <h1>TW2002: Redux</h1>

  <p>Because I'm not done playing yet.</p>
  <div class="game-menu">
    <button id="generate-game" on:click={newGalaxy}>
      Generate New Universe
    </button>
    <button id="generate-links" on:click={loadGalaxy}>Load Local Galaxy</button>
    <button id="start-game">Start Game</button>
    <div id="controls" class="section group">
      <div class="col">
              <label for="galSize">Galaxy size:
        <input type="number" value="10" name="galSize" id="galSize" /></label>
        <label for="warpsMin">Min warps:
        <input type="number" value="2" name="warpsMin" id="warpsMin" /></label>
        <label for="warpsMin">Max warps:
        <input type="number" value="6" name="warpsMax" id="warpsMax" /></label>
      </div>
    </div>

  </div>
<!-- {#if process.browser} -->
  <div class="sector-list">
    <div id="galaxy_info" />
  </div>
  <div class="sector-list">
    {#each galaxy as sector}
      <div class="svelte-universe">
        <span class="sector-details">
          Sector {sector.id}: {sector.name} |||
        </span>
        <span class="sector-warps">
          warps quota:
          <span class="warp">{sector.warpsQuota}</span>
          warps:
          {#each sector.outlinks as warp}
            <span class="warp">{warp}</span>
          {/each}
        </span>
      </div>
    {/each}
  </div>
  <!-- {/if} -->
</section>
