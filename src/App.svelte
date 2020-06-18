<script>
  import { onMount, onDestroy } from 'svelte';
  import Details from './Details.svelte';

  const second = 1000;
  const dataExpire = 14 * second;

  let data = [];
  let problemsOnly = true;
  let hosts = [];
  let services = [];
  let hostData = {};
  let lastUpdate = new Date(0);
  let sidebarHost = "";
  let sidebarService = "";
  async function update() {
    if ((new Date()) - lastUpdate > dataExpire) {
      return forceUpdate();
    }
  }
  async function forceUpdate() {
    return fetch(
      '/monitoring/list/servicegrid?format=json'
    ).then(r => {
      return r.json();
    }).then(d => {
      lastUpdate = new Date();
      data = d;
      reindex();
    });
  }
  async function reindex() {
    let hostSet = new Set();
    let serviceSet = new Set();
    hostData = {};
    for (let item of data) {
      if (!problemsOnly || item.service_state > 0) {
        if (!hostSet.has(item.host_name)) {
          hostData[item.host_name] = {};
        }
        hostSet.add(item.host_name);
        serviceSet.add(item.service_description);
        hostData[item.host_name][item.service_description] = item;
      }
    }
    hosts = Array.from(hostSet);
    hosts.sort();
    services = Array.from(serviceSet);
    services.sort();
  }
  async function hashchange() {
    let hash = window.location.hash.slice(1).split("/");
    sidebarHost = "";
    sidebarService = "";
    if (hash.length == 2) {
      sidebarHost = hash[0];
      sidebarService = hash[1];
    }
    update();
  }

  async function popup(host, service) {
    window.location.hash = "#" + host + "/" + service;
  }

  const states = ["ok", "warning", "critical", "unknown"];

  async function closeSidebar() {
    window.location.hash = "#";
    }

  onMount(hashchange);

  const interval = setInterval(update, dataExpire + second);
  onDestroy(() => clearInterval(interval));
</script>

<svelte:window on:hashchange={hashchange} />

<main>
  <p>Last update: {lastUpdate}</p>
  <button on:click={forceUpdate}>Force update</button>
  {#if !data}
  <p>No data has been received yet.</p>
  {/if}

  <input type=checkbox bind:checked={problemsOnly} on:change={reindex} />
  
  <div class="wrapper">
  <section>
    <table>
      <thead>
        <tr>
          <th />
          {#each services as service}
          <th>{service}</th>
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each hosts as host}
        <tr>
          <th>{host}</th>
          {#each services as service}
          {#if hostData[host][service]}
          <td class={states[hostData[host][service].service_state]} on:click={popup(host, service)}>
          </td>
          {:else}
          <td border="0" />
          {/if}
          {/each}
        </tr>
        {/each}
      </tbody>
    </table>
  </section>
  {#if hostData[sidebarHost]}
  {#if hostData[sidebarHost][sidebarService]}
  <Details item={hostData[sidebarHost][sidebarService]}>
    <button on:click={closeSidebar}>Close</button>
  </Details>
  {/if}
  {/if}
  </div>
</main>

<style>
	main {
    max-width: 1000px;
		margin: 0 auto;
	}

  .wrapper {
    display: flex;
    flex-direction: row;
    align-content: center;
    justify-content: center;
    width: 100%;
  }

  .wrapper > * {
		margin: 1em;
  }

  @media (max-width: 768px) {
    main {
      max-width: unset;
    }
  }

  table {
    font-size: 0.7em;
  }

  thead th {
    writing-mode: sideways-lr;
  }

  tbody td {
    min-width: 2em;
    min-height: 2em;
  }
  th, td {
    border: 0.2em solid black;
  }
</style>
