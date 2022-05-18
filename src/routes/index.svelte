<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState, basicSetup } from "@codemirror/basic-setup"
  import { EditorView, keymap } from "@codemirror/view"
  import { indentWithTab } from "@codemirror/commands"
  import { javascript } from "@codemirror/lang-javascript"
  import { ViewPlugin } from "@codemirror/view"
  import { minify } from "terser";

  let jsCode = "";
  let newScript = "";
  let scripts: { text: string, editable?: boolean }[] = [];
  let parent: HTMLDivElement

  const updatePlugin = ViewPlugin.fromClass(class {
		constructor() {}
		update(update) {
			if (update.docChanged) jsCode = update.state.doc.toString()
		}
	})

  $: if (jsCode && globalThis.localStorage) localStorage.setItem("code", jsCode)

  onMount(() => {

    jsCode = localStorage.getItem("code") || ""
    new EditorView({
      state: EditorState.create({
        doc: jsCode,
        extensions: [
          basicSetup,
          keymap.of([indentWithTab]),
          updatePlugin,
          javascript()
        ]
      }),
      parent
    })
  })
</script>

<div class="w-screen h-screen flex flex-row">
  <div class="text-center w-1/5 p-4">
    <h1 class="text-4xl font-bold mb-4">Scripts</h1>
    {#each scripts as script}
      {#if script.editable}
        <input class="text-center border-b border-gray-300" bind:value={script.text} on:blur={() => { script.editable = false}} placeholder="Edit Script"/>
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
  <div class="flex flex-col w-full">
    <div bind:this={parent} class="w-full"></div>
    {#await minify(jsCode) then minifiedCode}
      <div class="p-8 bg-[#f5f5f5]">
        <a href={"javascript:" + minifiedCode.code}>javascript:{minifiedCode.code}</a>
      </div>
    {/await}
  </div>
</div>