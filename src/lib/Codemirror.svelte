<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState, basicSetup } from "@codemirror/basic-setup"
  import { EditorView, keymap } from "@codemirror/view"
  import { indentWithTab } from "@codemirror/commands"
  import { javascript } from "@codemirror/lang-javascript"
  import { css } from "@codemirror/lang-css"
  import { ViewPlugin } from "@codemirror/view"

  export let language: "css" | "js";
  export let localStorageKey: string;
  export let code: string;
  let parent: HTMLDivElement;

  const updatePlugin = ViewPlugin.fromClass(class {
		constructor() {}
		update(update) {
			if (update.docChanged) code = update.state.doc.toString()
		}
	})

  $: if (code && globalThis.localStorage) localStorage.setItem(localStorageKey, code)

  onMount(() => {
    code = localStorage.getItem(localStorageKey) || ""
    new EditorView({
      state: EditorState.create({
        doc: code,
        extensions: [
          basicSetup,
          keymap.of([indentWithTab]),
          updatePlugin,
          language == "js" ? javascript() : css()
        ]
      }),
      parent
    })
  })
</script>

<div bind:this={parent}></div>