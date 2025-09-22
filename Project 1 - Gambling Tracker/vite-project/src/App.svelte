<script>
  import { onMount } from "svelte";
  import Chart from "./Chart.svelte";

  let username = "";
  let loggedIn = false;

  let bet = "";
  let game = "";
  let result = "";
  let date = "";
  let time = "";

  let entries = [];

  const getKey = (name) => `gamblingEntries_${name}`;

  function login() {
    if (!username.trim()) {
      alert("Type in your name to continue");
      return;
    }
    loggedIn = true;
    loadEntries();
  }

  function logout() {
    loggedIn = false;
    entries = [];
    username = "";
  }

  function loadEntries() {
    const saved = localStorage.getItem(getKey(username));
    entries = saved ? JSON.parse(saved) : [];
  }

  function saveEntries() {
    localStorage.setItem(getKey(username), JSON.stringify(entries));
  }

  function setDefaults() {
    const d = new Date();
    if (!date) {
      const m = String(d.getMonth() + 1).padStart(2, "0");
      const dd = String(d.getDate()).padStart(2, "0");
      date = `${d.getFullYear()}-${m}-${dd}`;
    }
    if (!time) {
      const h = String(d.getHours()).padStart(2, "0");
      const min = String(d.getMinutes()).padStart(2, "0");
      time = `${h}:${min}`;
    }
  }

  function addEntry() {
    if (!bet || !game.trim() || !result || !date || !time) {
      alert("Fill out everything please");
      return;
    }

    const entry = {
      id: Date.now() + Math.random(),
      bet: Number(bet),
      game,
      result: Number(result),
      date,
      time,
      createdAt: Date.now(),
    };

    entries = [...entries, entry].sort((a, b) => b.createdAt - a.createdAt);
    saveEntries();

    bet = "";
    game = "";
    result = "";
    date = "";
    time = "";
    setDefaults();
  }

  function deleteEntry(id) {
    entries = entries.filter((e) => e.id !== id);
    saveEntries();
  }

  $: labels = entries.map((e) => `${e.date} ${e.time}`).reverse();
  $: balances = entries.map((e) => e.result).reverse();

  $: totalBets = entries.length;
  $: totalWon = entries
    .filter((e) => e.result > 0)
    .reduce((s, e) => s + e.result, 0);
  $: totalLost = entries
    .filter((e) => e.result < 0)
    .reduce((s, e) => s + e.result, 0);
  $: netBalance = totalWon + totalLost;

  onMount(() => {
    if (loggedIn) {
      loadEntries();
      entries.sort((a, b) => b.createdAt - a.createdAt);
      setDefaults();
    }
  });
</script>

<main>
  {#if !loggedIn}
    <div class="login-box">
      <h2>Login with your name</h2>
      <input type="text" bind:value={username} placeholder="Enter your name" />
      <button on:click={login}>Login</button>
    </div>
  {:else}
    <header class="user-header">
      <h2>Hey, {username}</h2>
      <button on:click={logout}>Logout</button>
    </header>

    <section class="stats">
      <p>Total Bets: {totalBets}</p>
      <p>Total Won: <span class="won">${totalWon}</span></p>
      <p>Total Lost: <span class="lost">{totalLost}</span></p>
      <p>Net Balance: {netBalance >= 0 ? `+${netBalance}` : `${netBalance}`}</p>
    </section>

    <form on:submit|preventDefault={addEntry}>
      <input type="number" bind:value={bet} placeholder="Bet size" required />
      <input
        type="text"
        bind:value={game}
        placeholder="What was it on?"
        required
      />
      <input
        type="number"
        bind:value={result}
        placeholder="Win/Loss (+/-)"
        required
      />
      <input type="date" bind:value={date} required />
      <input type="time" bind:value={time} required />
      <button type="submit">Log Bet</button>
    </form>

    {#if entries.length === 0}
      <p>No entries yet.</p>
    {:else}
      <div id="entries">
        {#each entries as e (e.id)}
          <div class="entry">
            <h4>{e.date} @ {e.time}</h4>
            <p>Bet: ${e.bet} on {e.game}</p>
            <p>
              Result:
              {#if e.result >= 0}
                <span class="won">+${e.result}</span>
              {:else}
                <span class="lost">{e.result}</span>
              {/if}
            </p>
            <button on:click={() => deleteEntry(e.id)}>Delete</button>
          </div>
        {/each}
      </div>
    {/if}

    <h2>Net Balance</h2>
    {#if entries.length > 0}
      <Chart {labels} {balances} />
    {:else}
      <p>No data yet</p>
    {/if}

    <footer class="disclaimer">
      Gambling can be addictive and lead to serious financial and personal
      problems if not done responsibly. This site is intended to help you stay
      aware of your bets, monitor wins and losses, and make sure you aren’t
      losing more than you can afford. Please remember to gamble for
      entertainment only and set strict limits to protect your well-being. If
      you or someone you know is struggling with problem gambling, call the
      National Problem Gambling Helpline at <strong>1-800-522-4700</strong> for free,
      confidential support available 24/7. This statement was generate by Chatgpt. 
    </footer>
  {/if}
</main>

<style>
  main {
    max-width: 600px;
    margin: auto;
    padding: 1rem;
    font-family: sans-serif;
    color: white;
  }
  form {
    display: grid;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }
  input,
  button {
    padding: 0.5rem;
    font-size: 1rem;
  }
  .entry {
    border: 1px solid #444;
    padding: 0.5rem;
    margin: 0.5rem 0;
    border-radius: 6px;
    background: #222;
  }
  .stats {
    background: #222;
    padding: 0.5rem;
    margin-bottom: 1rem;
    border-radius: 6px;
  }
  .won {
    color: #22c55e;
  }
  .lost {
    color: #ef4444;
  }
  h2 {
    margin-top: 2rem;
    text-align: center;
  }
  .login-box {
    text-align: center;
    margin-top: 4rem;
  }
  .user-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
  }
  .disclaimer {
    font-size: 0.7rem;
    color: #aaa;
    text-align: center;
    margin-top: 2rem;
  }
  .disclaimer strong {
    color: #ef4444;
  }
</style>
