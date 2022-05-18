<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState, basicSetup } from "@codemirror/basic-setup"
  import { EditorView, keymap } from "@codemirror/view"
  import { indentWithTab } from "@codemirror/commands"
  import { javascript } from "@codemirror/lang-javascript"
  import { css } from "@codemirror/lang-css"
  import { ViewPlugin } from "@codemirror/view"
  import { showPanel, type Panel } from "@codemirror/view"
  import { StateField, StateEffect } from "@codemirror/state"

  export let language: "css" | "js";
  export let localStorageKey: string;
  export let code: string;
  export let title: string;
  let parent: HTMLDivElement;

  const helpTheme = EditorView.baseTheme({
    ".cm-help-panel": {
      padding: "5px 10px",
      backgroundColor: "#fffa8f",
      fontFamily: "monospace"
    }
  })

  function wordCountPanel(view: EditorView): Panel {
    let dom = document.createElement("div")
    dom.textContent = "Test"
    return {
      dom
    }
  }

  const updatePlugin = ViewPlugin.fromClass(class {
		constructor() {}
		update(update) {
			if (update.docChanged) code = update.state.doc.toString()
		}
	})

  $: if (code && globalThis.localStorage) localStorage.setItem(localStorageKey, code)

  onMount(() => {
    code = localStorage.getItem(localStorageKey) || ""
    const view = new EditorView({
      state: EditorState.create({
        doc: code,
        extensions: [
          basicSetup,
          keymap.of([indentWithTab]),
          updatePlugin,
          language == "js" ? javascript() : css(),
          helpTheme
        ]
      }),
      parent
    });

    view.dispatch({
      effects: toggleHelp.of(!view.state.field(helpPanelState))
    })
  })
</script>

<div bind:this={parent}></div>