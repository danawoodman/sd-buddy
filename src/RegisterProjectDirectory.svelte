<script lang="ts">
  import { open as openDialog } from "@tauri-apps/api/dialog";
  import { appDir } from "@tauri-apps/api/path";
  import { open } from "@tauri-apps/api/shell";
  import { onMount } from "svelte";
  import { get, set } from "tauri-settings";
  import { stableDiffusionDirectory } from "./store";

  type directory = {
    stableDiffusionDirectory: string;
  };

  let stableDiffusionDirectoryInput: HTMLInputElement;
  let stableDiffusionOutputDirectory: string = "";

  // Flags
  let isReregistering: boolean = false; // re-registering the Stable Diffusion directory

  $: stableDiffusionDirectoryIsRegistered =
    stableDiffusionDirectoryInput &&
    stableDiffusionDirectoryInput.value.trim() !== "";

  $: {
    if ($stableDiffusionDirectory) {
      stableDiffusionOutputDirectory = `${$stableDiffusionDirectory}/outputs/txt2img-samples`;
    }
  }

  async function openDirectorySelectionDialog() {
    // Open a selection dialog for directories
    const selected = await openDialog({
      directory: true,
      multiple: false,
      defaultPath: await appDir(),
    });
    if (Array.isArray(selected)) {
      // user selected multiple directories
    } else if (selected === null) {
      // user cancelled the selection
    } else {
      // user selected a single directory
      stableDiffusionDirectoryInput.value = selected;
    }
  }

  async function openDirectory(directory: string) {
    await open(`file://${directory}`);
  }

  async function saveStableDiffusionDirectory() {
    stableDiffusionDirectory.set(stableDiffusionDirectoryInput.value);
    stableDiffusionDirectoryInput.value = "";
    isReregistering = false;

    // Saves to /Users/your.user/Library/Application Support/com.sd-buddy.breadthe/settings.json
    await set<directory>("stableDiffusionDirectory", $stableDiffusionDirectory)
      .then(() => console.log("Stable Diffusion directory saved"))
      .catch((err) => console.log(err));
  }

  onMount(async () => {
    await get<directory>("stableDiffusionDirectory")
      .then((directory) => {
        stableDiffusionDirectory.set(directory);
      })
      .catch((err) => {
        // do nothing, the user will have to set the directory
      });
  });
</script>

<div class="flex flex-col gap-2">
  {#if !$stableDiffusionDirectory || isReregistering}
    <div class="flex gap-2 mb-4">
      <input
        bind:this={stableDiffusionDirectoryInput}
        type="text"
        placeholder="/absolute/path/to/Stable/Diffusion/directory"
        class="flex-1"
      />

      <button
        class=""
        title="Browse directories"
        on:click={openDirectorySelectionDialog}>Browse</button
      >

      <button
        class=""
        title="Register Stable Diffusion directory"
        disabled={!stableDiffusionDirectoryIsRegistered}
        on:click={saveStableDiffusionDirectory}>Save</button
      >
    </div>
  {/if}
  {#if $stableDiffusionDirectory}
    <div class="flex flex-col gap-1">
      <div class="flex items-center gap-2">
        <span class="text-xs font-bold">SD project:</span>
        <a
          href={`file://${$stableDiffusionDirectory}`}
          class="font-mono text-xs text-blue-600 hover:underline"
          on:click|preventDefault={() =>
            openDirectory($stableDiffusionDirectory)}
          >{$stableDiffusionDirectory}</a
        >
        <button
          title="Re-register the Stable Diffusion directory"
          class="transparent"
          on:click={() => (isReregistering = !isReregistering)}
          ><svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-4 h-4"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0013.803-3.7M4.031 9.865a8.25 8.25 0 0113.803-3.7l3.181 3.182m0-4.991v4.99"
            />
          </svg>
        </button>
      </div>
      <div class="flex items-center gap-2">
        <span class="text-xs font-bold">SD output:</span>
        <a
          href={`file://${stableDiffusionOutputDirectory}`}
          class="font-mono text-xs text-blue-600 hover:underline"
          on:click|preventDefault={() =>
            openDirectory(stableDiffusionOutputDirectory)}
          >{stableDiffusionOutputDirectory}</a
        >
      </div>
    </div>
  {/if}
</div>

<style>
</style>
