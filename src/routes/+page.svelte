<h1 style="text-align: center;">Book Fuzzy Finder</h1>
<p style="text-align: center;"> A simple fuzzy finder for book pages based on quoted snippets. </p>

<!-- This is a Svelte component for a book fuzzy finder -->
<script>
   let input = '';
   let results = [];

function extractPageNumber(previewLink) {
  try {
    const url = new URL(previewLink);
    const pgParam = url.searchParams.get('pg'); // e.g. "PA220" or "PT1510"
    if (!pgParam) return null;

    // Remove any prefix letters like "PA" or "PT"
    const pageNum = pgParam.replace(/^[A-Z]+/, '');

    return pageNum;
  } catch (e) {
    return null;
  }
}
   $: if (input.trim()) {
      const phrase = `"${input}"`;
      fetch(`https://www.googleapis.com/books/v1/volumes?q=${encodeURIComponent(phrase)}&printType=books&maxResults=3`)
         .then(res => res.json())
         .then(data => {
             results = data.items ? data.items.map(item =>
             ({
            title: item.volumeInfo.title,
            author: item.volumeInfo.
            authors?.join(', ') ?? 'Unknown Author',
            snippet: item.searchInfo?.textSnippet ?? 'Snippet not available',
            link: item.volumeInfo.previewLink
         })) : [];
      })
      .catch(err => {
         console.error('Error fetching data:', err);
         results = [];
      });
   } else {
      results = [];
   }
</script>

<!-- CSS -->
<style>
  :root {
  color-scheme: light dark;
  --bg-color: #f0f2f5;
  --text-color: #1a1a1a;
  --border-color: #ccc;
  --item-bg: #fff;
  --item-hover-bg: #e6f0ff;
  --item-border: #eee;
  --input-bg: #fff;
  --input-border: #ccc;
  --link-color: #0077cc;
  --muted-color: #666;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg-color: #1e1e1e;
    --text-color: #f2f2f2;
    --border-color: #444;
    --item-bg: #2a2a2a;
    --item-hover-bg: #333;
    --item-border: #3a3a3a;
    --input-bg: #2b2b2b;
    --input-border: #555;
    --link-color: #4ea1ff;
    --muted-color: #aaa;
  }
}

h1, p {
  text-align: center;
  margin-bottom: 1rem;
}

.input-container {
  max-width: 600px;
  margin: 2rem auto 1rem;
}

.input-container input {
  width: 100%;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  background: var(--input-bg);
  color: var(--text-color);
  border: 1px solid var(--input-border);
  border-radius: 8px;
  box-shadow: 0 1px 2px rgba(0,0,0,0.05);
  box-sizing: border-box;
  transition: border 0.2s;
}

.input-container input:focus {
  outline: none;
  border-color: var(--link-color);
}

.command-palette {
  list-style: none;
  padding: 0;
  margin: 1rem auto;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  max-width: 600px;
  background: var(--item-bg);
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.command-palette li {
  padding: 1rem;
  border-bottom: 1px solid var(--item-border);
  cursor: pointer;
  transition: background 0.2s;
}

.command-palette li:hover {
  background: var(--item-hover-bg);
}

.command-palette li:last-child {
  border-bottom: none;
}

.command-card:link,
.command-card:visited,
.command-card:hover,
.command-card:active {
  text-decoration: none;
  color: inherit;
}

a {
  color: var(--link-color);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

small, em {
  color: var(--muted-color);
}
</style>

<!-- DOM -->
<main><div class="input-container">
  <input type="text" bind:value={input} placeholder="🔍  Search for a snippet..." />
</div>
{#if results.length}
  <ul class="command-palette">
    {#each results as r}
      <li>
        <a
          href={`${r.link}&q=${encodeURIComponent(r.link)}`}
          target="_blank"
          class="command-card"
        >
          <div class="command-content">
            <strong>{r.title}</strong><br />
            <em>{r.author}</em><br />
            <small>{@html r.snippet}</small><br />
            <small>
              {#if extractPageNumber(r.link)}
                Page: {extractPageNumber(r.link)}
              {:else}
                Page number not available.
              {/if}
            </small>
          </div>
        </a>
      </li>
    {/each}
  </ul>
{/if}
</main>