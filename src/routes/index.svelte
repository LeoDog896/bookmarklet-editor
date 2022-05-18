<script lang="ts">
  import { minify } from "terser";

  let jsCode: string = "";
</script>

<input bind:value={jsCode}/>

{#await minify(jsCode) then minifiedCode}
  <p>{minifiedCode.code}</p>
{/await}