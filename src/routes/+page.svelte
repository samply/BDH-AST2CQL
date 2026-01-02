<script lang="ts">
  export const ssr = false;

  if (import.meta.hot) {
    import.meta.hot.on("vite:beforeUpdate", () => {
      window.location.reload();
    });
  }

  import "@samply/lens";
  import { translateAstToCql } from "$lib/ast-to-cql-translator";
  import { measures } from "$lib/measures";
  import catalogueProd from "$lib/catalog.json";

  import {
  buildLibrary,
    buildMeasure,
    getAst,
    setCatalogue,
    setQueryStore,
    type AstTopLayer,
    type Catalogue,
    type QueryItem,
  } from "@samply/lens";

  const catalogue = catalogueProd as Catalogue;
  setCatalogue(catalogue);

  let inputAst = "";
  let inputQuery = "";
  let outputValue = "";

  let errorMessage = "";
  let successMessage = "";

  function parseQueryFromUrl(url: string): QueryItem[][] {
    const params = new URL(url).searchParams;
    const queryParam = params.get("query");

    if (!queryParam) {
      throw new Error("URL does not contain a valid query parameter.");
    }

    return JSON.parse(atob(queryParam));
  }

  function parseAstFromInput(ast: string): AstTopLayer {
    return JSON.parse(ast);
  }

  const handleButtonClick = () => {
    errorMessage = "";
    successMessage = "";
    outputValue = "";

    if (!inputAst.trim() && !inputQuery.trim()) {
      errorMessage = "Either a URL or an AST JSON must be provided.";
      return;
    }

    let parsedAst: AstTopLayer;

    try {
      if (inputQuery.trim()) {
        const parsedQuery = parseQueryFromUrl(inputQuery);
        setQueryStore(parsedQuery);

        parsedAst = getAst();
        inputAst = JSON.stringify(parsedAst, null, 2);
      } else {
        parsedAst = parseAstFromInput(inputAst);
      }

      let cql = translateAstToCql(
        parsedAst,
        false,
        "DKTK_STRAT_DEF_IN_INITIAL_POPULATION",
        measures
      );

        const lib = buildLibrary(cql);
    const measure = buildMeasure(
      lib.url,
      measures.map((m) => m.measure),
    );

    outputValue = btoa(
      JSON.stringify({
        lang: "cql",
        lib,
        measure,
      }),
    );

      successMessage = "Conversion successful.";
    } catch (err) {
      console.error(err);
      errorMessage =
        err instanceof Error
          ? err.message
          : "Conversion failed. Please check your inputs.";
    }
  };

  const clearAll = () => {
    inputAst = "";
    inputQuery = "";
    outputValue = "";
    errorMessage = "";
    successMessage = "";
  };
</script>

<div class="container">
  <div class="card">
    <h1>BioDataHub Lens-AST → CQL</h1>

    <p class="description">
      Paste a URL from the DKTK/CCP Explorer or provide a raw AST JSON to generate
      the corresponding CQL.
    </p>

    <div class="form-group">
      <label>URL (CCP Explorer)</label>
      <input
        type="text"
        bind:value={inputQuery}
        placeholder="Paste the URL containing the query parameter"
      />
    </div>

    <div class="form-group">
      <label>Input AST (JSON)</label>
      <textarea
        rows="5"
        bind:value={inputAst}
        placeholder="Paste AST JSON here..."
      />
      <small class="hint">
        Provide either a CCP Explorer URL or a raw AST JSON.
      </small>
    </div>

    <button
      on:click={handleButtonClick}
      disabled={!inputAst.trim() && !inputQuery.trim()}
    >
      Convert
    </button>

    <button class="secondary" on:click={clearAll}>
      Clear
    </button>

    {#if errorMessage}
      <div class="message error">{errorMessage}</div>
    {/if}

    {#if successMessage}
      <div class="message success">{successMessage}</div>
    {/if}

    <div class="form-group">
      <label>Output CQL</label>
      <textarea
        rows="6"
        readonly
        value={outputValue}
        placeholder="Result will appear here..."
      />
    </div>
  </div>
</div>

<footer>
  <p class="disclaimer">
    This software is a lightweight, client-side tool intended solely for local
    use in a web browser. No guarantees are made regarding correctness,
    completeness, or suitability for any particular purpose. The application is
    provided “as is” without any warranty, express or implied. The authors assume
    no liability for any damages resulting from the use of this software.
  </p>

  <nav>
    <a href="https://dktk.dkfz.de/ccp" target="_blank">
      Clinical Communication Platform (CCP)
    </a>
    <a href="https://hub.dkfz.de/s/MPCg2kK23LH8Yii" target="_blank">
      Nutzungsvereinbarung
    </a>
    <a href="https://hub.dkfz.de/s/5LPccy6fgRSoD65" target="_blank">
      Datenschutz
    </a>
    <a href="https://www.dkfz.de/de/impressum.html" target="_blank">
      Impressum
    </a>
  </nav>
</footer>

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
    max-width: 36rem;
  }

  h1 {
    font-size: 1.5rem;
    margin-bottom: 1rem;
  }

  .description {
    font-size: 0.9rem;
    color: #4b5563;
    margin-bottom: 1rem;
  }

  .form-group {
    margin-bottom: 1rem;
  }

  label {
    font-size: 0.875rem;
    font-weight: 500;
    margin-bottom: 0.25rem;
    display: block;
  }

  input,
  textarea {
    width: 100%;
    padding: 0.5rem 0.75rem;
    border: 1px solid #d1d5db;
    border-radius: 0.375rem;
    font-family: inherit;
  }

  textarea {
    resize: vertical;
    background-color: #f9fafb;
  }

  .hint {
    font-size: 0.75rem;
    color: #6b7280;
  }

  button {
    width: 100%;
    background-color: #3b82f6;
    color: white;
    padding: 0.6rem;
    border-radius: 0.375rem;
    border: none;
    cursor: pointer;
    margin-bottom: 0.5rem;
  }

  button.secondary {
    background-color: #6b7280;
  }

  button:disabled {
    background-color: #9ca3af;
    cursor: not-allowed;
  }

  .message {
    padding: 0.5rem;
    border-radius: 0.375rem;
    margin-bottom: 0.75rem;
    font-size: 0.875rem;
  }

  .message.error {
    background-color: #fee2e2;
    color: #991b1b;
  }

  .message.success {
    background-color: #dcfce7;
    color: #166534;
  }

  footer {
    margin-top: 1.5rem;
    text-align: center;
    font-size: 0.75rem;
    color: #6b7280;
  }

  footer nav {
    margin-top: 0.5rem;
  }

  footer a {
    margin: 0 0.5rem;
    color: #3b82f6;
    text-decoration: none;
  }

  footer a:hover {
    text-decoration: underline;
  }
</style>
