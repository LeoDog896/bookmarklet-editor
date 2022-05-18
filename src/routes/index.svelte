<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState, basicSetup } from "@codemirror/basic-setup"
  import { EditorView, keymap } from "@codemirror/view"
  import { indentWithTab } from "@codemirror/commands"
  import { javascript } from "@codemirror/lang-javascript"
  import { minify } from "terser";

  let jsCode: string = "";
  let parent: HTMLDivElement

  onMount(() => {
    new EditorView({
      state: EditorState.create({
        jsCode,
        extensions: [
          basicSetup,
          keymap.of([indentWithTab]),
          javascript()
        ]
      }),
      parent
    })
  })
</script>
<div bind:this={parent}></div>

{#await minify(jsCode) then minifiedCode}
  <a href={"javascript:" + minifiedCode}>javascript:{minifiedCode.code}</a>
{/await}