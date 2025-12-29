<script lang="ts">
  export const ssr = false;

    // Using hot module replacement (HMR) with custom elements (aka web
    // components) does not work because a custom element cannot be updated once
    // registered, see https://github.com/WICG/webcomponents/issues/820.
    // Therefore we do a full page reload instead of HMR.
    if (import.meta.hot) {
        import.meta.hot.on("vite:beforeUpdate", () => {
            window.location.reload();
        });
    }
    import "@samply/lens";

  import { translateAstToCql } from "$lib/ast-to-cql-translator";
  import { measures } from "$lib/measures";

  import {buildLibrary, buildMeasure} from "@samply/lens"

  let inputValue = '';
  let outputValue = '';



  const handleButtonClick = () => {
        const lib = buildLibrary(inputValue);

  const measure = buildMeasure(
    lib.url,
    measures.map((m) => m.measure),
  );

    JSON.parse(inputValue)

    const result = translateAstToCql(JSON.parse(inputValue), true, "DKTK_STRAT_DEF_IN_INITIAL_POPULATION", measures);
    outputValue = result;
  };
</script>

<div class="container">
  <div class="card">
    <h1>BioDataHub Lens-AST 2 CQL</h1>
    
    <div class="form-group">
      <label>Input:</label>
      <input type="text" bind:value={inputValue} placeholder="Enter some text..." />
    </div>

    <button on:click={handleButtonClick}>Process Input</button>

    <div class="form-group">
      <label>Output:</label>
      <textarea value={outputValue} readonly rows="4" placeholder="Result will appear here..."></textarea>
    </div>
  </div>
</div>

<style>
  .container {
    min-height: 100vh;
    background-color: #f3f4f6;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem;
  }

  .card {
    background-color: white;
    border-radius: 0.5rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 1.5rem;
    width: 100%;
    max-width: 28rem;
  }

  h1 {
    font-size: 1.5rem;
    font-weight: bold;
    margin-bottom: 1.5rem;
    color: #1f2937;
  }

  .form-group {
    margin-bottom: 1rem;
  }

  label {
    display: block;
    font-size: 0.875rem;
    font-weight: 500;
    color: #374151;
    margin-bottom: 0.5rem;
  }

  input, textarea {
    width: 100%;
    padding: 0.5rem 0.75rem;
    border: 1px solid #d1d5db;
    border-radius: 0.375rem;
    font-family: inherit;
  }

  input:focus, textarea:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }

  textarea {
    background-color: #f9fafb;
    color: #374151;
    resize: vertical;
  }

  button {
    width: 100%;
    background-color: #3b82f6;
    color: white;
    font-weight: 500;
    padding: 0.5rem 1rem;
    border-radius: 0.375rem;
    border: none;
    cursor: pointer;
    margin-bottom: 1rem;
    transition: background-color 0.2s;
  }

  button:hover {
    background-color: #2563eb;
  }
</style>