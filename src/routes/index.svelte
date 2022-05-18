<script lang="ts">
  import { minify } from "terser";
  import Codemirror from "$lib/Codemirror.svelte"

  const cssCodeGen = (content: string) => content ? `document.head.appendChild(document.createElement("style")).innerHTML = \`${content}\`;` : ""

  let jsCode = "";
  let cssCode = "";
  let cssAfterScript = false;
  let newScript = "";
  let scripts: { text: string, editable?: boolean }[] = [];

  $: appendedCode = cssAfterScript ? (jsCode + cssCodeGen(cssCode)) : (cssCodeGen(cssCode) + jsCode)

</script>

<div class="w-screen h-screen flex flex-row">
  <div class="text-center w-1/5 p-4">
    <div>
      <h1 class="text-4xl font-bold mb-4">Scripts</h1>
      {#each scripts as script}
        {#if script.editable}
          <input class="text-center border-b bordercssAfterScript-gray-300" bind:value={script.text} on:blur={() => { script.editable = false}} placeholder="Edit Script"/>
        {:else}
          <p class="inline" on:dblclick={() => { script.editable = true }}>{script.text}</p>
          <button class="p-2 rounded-full inline bg-red-400" on:click={() => {
            scripts.splice(scripts.indexOf(script), 1);
            scripts = scripts;
          }}></button>
          <br/>
        {/if}
      {/each}
      <input class="mt-4 text-center border-b border-gray-300 w-full" bind:value={newScript} on:keypress={e => {
        if (e.key == "Enter" && newScript) {
          scripts = [...scripts, { text: newScript }];
          newScript = ""
        }
      }} placeholder="Add Script"/>
    </div>
    <div>
      <h1 class="text-4xl font-bold mb-4 mt-4">Options</h1>
      <input type="checkbox" bind:checked={cssAfterScript}><span class="ml-1">CSS After Script</span>
    </div>
  </div>
  <div class="flex flex-col w-full">
    <Codemirror language={"js"} title={"JavaScript"} bind:code={jsCode} localStorageKey={"code"}/>
    <Codemirror language={"css"} title={"CSS"} bind:code={cssCode} localStorageKey={"css"}/>
    {#await minify(appendedCode) then minifiedCode}
      <div class="p-8 bg-[#f5f5f5]">
        <a href={"javascript:" + minifiedCode.code}>javascript:{minifiedCode.code}</a>
      </div>
    {/await}
  </div>
</div>