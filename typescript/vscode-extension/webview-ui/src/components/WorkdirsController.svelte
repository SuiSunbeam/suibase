<script lang="ts">
  import { VSCode } from "../lib/VSCode";
  import { SuibaseJSONStorage } from "../common/SuibaseJSONStorage";

  interface WorkdirData {
    name: string;
    status: string;
  }

  export let workdirs: WorkdirData[] = [
    { name: "Localnet", status: "Running" },
    { name: "Devnet", status: "Degraded" },
    { name: "Testnet", status: "Stopped" },
    { name: "Mainnet", status: "Down" },
  ];

  function handleStartClick(workdir: WorkdirData) {
    // let workdir=workdir.name

    VSCode.postMessage({
      command: "start",
      workdir: workdir.name,
    });
    console.log("handleStartClick called");

    // Do a POST request equivalent to http://0.0.0.0:44399 with:
    // header is Content-Type: application/json
    // body is {"id":1,"jsonrpc":"2.0","method":"getLinks","params":{"workdir":"workdir.name"}}
    const url = "http://localhost:44399";
    const headers = {
      "Content-Type": "application/json",
    };
    const body = {
      id: 1,
      jsonrpc: "2.0",
      method: "getLinks",
      params: {
        workdir: workdir.name,
      },
    };

    fetch(url, {
      method: "POST",
      headers: headers,
      body: JSON.stringify(body),
    })
      .then((response) => {
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        return response.json();
      })
      .then((data) => {
        console.log(data);
      })
      .catch((error) => {
        console.error("There was a problem with the fetch operation:", error);
      });
  }

  function handleStopClick(workdir: WorkdirData) {
    VSCode.postMessage({
      command: "stop",
      workdir: workdir.name,
    });
  }

  function handleRegenClick(workdir: WorkdirData) {
    VSCode.postMessage({
      command: "regen",
      workdir: workdir.name,
    });
  }
</script>

<main>
  <!---
  <vscode-dropdown>
    <vscode-option>Option Label #1</vscode-option>
    <vscode-option>Option Label #2</vscode-option>
    <vscode-option>Option Label #3</vscode-option>
  </vscode-dropdown>

  <vscode-button appearance="primary">Button Text</vscode-button>
  <vscode-button appearance="secondary">Button Text</vscode-button>

  <vscode-button appearance="icon" aria-label="Confirm">
    <span class="codicon codicon-check" />
  </vscode-button>
-->
  {#each workdirs as workdir}
    <div class="workdir_row">
      <h2 class="workdir">{workdir.name}</h2>
      <h2 class="status">{workdir.status}</h2>
      {#if workdir.status === "Stopped"}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <vscode-button on:click={() => handleStartClick(workdir)}>
          Start
          <span slot="start" class="codicon codicon-debug-start" />
        </vscode-button>
      {:else}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <vscode-button on:click={() => handleStopClick(workdir)}>
          Stop
          <span slot="start" class="codicon codicon-debug-stop" />
        </vscode-button>
      {/if}
      {#if workdir.name === "Localnet"}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <vscode-button on:click={() => handleRegenClick(workdir)}>
          Regen
          <span slot="start" class="codicon codicon-refresh" />
        </vscode-button>
      {/if}
    </div>
  {/each}
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: flex-start;
  }

  div {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
    width: 400px;
  }

  div.workdir_row {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
    width: 310px;
    gap: 10px;
    padding: 3px 0px;
    border: 1px solid gray;
    border-radius: 5px;
  }

  h2.workdir {
    margin: 0;
    font-size: 16px;
    width: 30px;
    text-align: left;
    padding-left: 5px;
  }

  h2.status {
    margin: 0;
    font-size: 9px;
    color: #666;
    width: 70px;
    text-align: right;
  }
</style>
