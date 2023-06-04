<script>
  import { onMount } from "svelte";

  const players = new Map([
    ["AmbassadorMowgli", "7478"],
    ["Ayanda", "8574"],
    ["Little One", "diff"],
    ["MrCheeseF", "EUW"],
    ["DarkstarEX274", "5921"],
    ["Mingus Khan", "7139"],
    ["UncleRuckus", "naai"],
    ["johannes", "Panda"],
    ["ProlificPotato", "5717"],
    ["SyberNinja", "EUW"],
    ["RobertFreeman", "1492"],
  ]);

  let playerData = new Map();

  /**
   * Function to make API requests
   * @param {string} name
   * @param {string} tag
   */
  async function makeAPIRequest(name, tag) {
    const url = `https://api.henrikdev.xyz/valorant/v1/mmr-history/eu/${name}/${tag}`;
    const response = await fetch(url);
    const data = await response.json();
    playerData = new Map(playerData);
    playerData.set(name, data.data);

    // Setting the mmrGain by using ranking_in_tier as the storage because im lazy
    let mmrGain = 0;
    for (let i = 0; i < data.data.length; i++) {
      mmrGain = mmrGain + data.data[i].mmr_change_to_last_game;
    }
    playerData.get(name)[0].ranking_in_tier = mmrGain;

    // Setting losing streak using date_raw as the storage beacause im lazy again
    let losingStreak = 0;
    while (data.data[losingStreak].mmr_change_to_last_game < 0) {
      losingStreak++;
    }

    playerData.get(name)[0].date_raw = losingStreak;
  }

  function displayPlayerData() {
    console.log(playerData);
  }

  // Sorting player data based on elo for the leaderboard
  function sortingPlayerData() {
    const array = Array.from(playerData);
    array.sort((a, b) => b[1][0].elo - a[1][0].elo);
    playerData = new Map(array);
  }

  onMount(async () => {
    // Loop over the players map and make API requests for each name and tag
    for (const [name, tag] of players) {
      await makeAPIRequest(name, tag);
    }
    sortingPlayerData();
    displayPlayerData();
  });
</script>

<main>
  {#if playerData.size > 0}
    <body class="table-center">
      <table class="styled-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>MMR Change</th>
            <th>Rank</th>
            <th>Loss Streak</th>
          </tr>
        </thead>
        <tbody>
          {#each Array.from( playerData, ([name, data]) => ({ name, data }) ) as { name, data }}
            <tr>
              <td>{name}</td>
              <td>{data[0].ranking_in_tier}</td>
              <td
                ><img
                  src={data[0].images.small}
                  alt={data[0].currenttierpatched}
                /></td
              >
              <td>{data[0].date_raw}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    </body>
  {:else}
    <p>Loading...</p>
  {/if}
</main>

<style>
  img {
    max-width: 100%;
    height: auto;
  }

  .table-center {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  main {
    flex: 1;
    display: flex;
    flex-direction: column;
    padding: 1rem;
    width: 100%;
    max-width: 64rem;
    margin: 0 auto;
    box-sizing: border-box;
  }

  .styled-table {
    border-collapse: separate;
    border-spacing: 0 3px;
    margin: 25px;
    font-size: 0.9em;
    font-family: sans-serif;
    min-width: 400px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
  }

  .styled-table thead tr {
    background-color: #343a3f;
    color: #ffffff;
    text-align: left;
  }

  .styled-table th,
  .styled-table td {
    padding: 20px;
    padding-left: 100px;
    padding-right: 100px;
  }

  .styled-table tbody tr {
    border-bottom: 1px solid #343a3f;
    background-color: #343a3f;
    font-weight: bold;
    color: #f3f3f3;
    white-space: nowrap;
  }

  .styled-table tbody tr:last-of-type {
    border-bottom: 2px solid #f3f3f3;
  }
</style>
