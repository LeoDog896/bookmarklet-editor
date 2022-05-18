<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState, basicSetup } from "@codemirror/basic-setup"
  import { EditorView, keymap } from "@codemirror/view"
  import { indentWithTab } from "@codemirror/commands"
  import { javascript } from "@codemirror/lang-javascript"
  import { ViewPlugin } from "@codemirror/view"
  import { minify } from "terser";

  let jsCode: string = "";
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
<div bind:this={parent}></div>

{#await minify(jsCode) then minifiedCode}
  <a href={"javascript:" + minifiedCode.code}>javascript:{minifiedCode.code}</a>
{/await}