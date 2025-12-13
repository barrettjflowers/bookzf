<div class="header">
<h1 style="text-align: center; margin-bottom: 1.5rem; margin-top: 0rem;">Bookzf</h1>
<p style="margin: 0.03rem 0; text-align: center;"> Do you switch between audiobooks and physical books? Bookzf can help you find the page you're looking for. </p>
<p style="text-align: center;"> A fuzzy finder for book pages. </p>
</div>

<script>
   let input = '';
   let results = [];

function extractPageNumber(previewLink) {
  try {
    const url = new URL(previewLink);
    const pgParam = url.searchParams.get('pg');
    if (!pgParam) return null;
    const pageNum = pgParam.replace(/^[A-Z]+/, '');

    return pageNum;
  } catch (e) {
    return null;
  }
}

let hasResults = false;
   $: if (input.trim()) {
	  let hasResults = false;
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
      hasResults = true;
      results = [];
   }
</script>
<style>
  :root {
  --bg-color: #ede9df;
  --text-color: #1a1a1a;
  --border-color: transparent;
  --item-bg: #e3dcca;
  --item-hover-bg: #e6f0ff;
  --item-border: #eee;
  --input-bg: #e3dcca;
  --input-border: #ccc;
  --link-color: #0077cc;
  --muted-color: #666;

	font-family: menlo;
  background-color: var(--bg-color);
}

.header {
top: 0;
position: sticky;
padding: 1em;
background-color: var(--input-bg);
width: 100vw;
height: 100px;
margin-left: calc(50% - 50vw);
margin-top: -1rem;
}

.header p {
margin: 0.3rem 0;
}

h1, p {
  text-align: center;
  margin-bottom: 1rem;
}

.input-container {
  max-width: 600px;
  margin: 2rem auto 1rem;
	margin-top: 1.5rem;
	margin-bottom: 0rem;
}

.input-container input {
  width: 100%;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  background: var(--input-bg);
  color: var(--text-color);
  border: none;
  box-sizing: border-box;
  transition: border 0.2s;
}

.input-container input:focus {
  outline: none;
}

.command-palette {
  list-style: none;
  padding: 0;
  margin: 1rem auto;
	margin-top: 0rem;
  border: 1px solid var(--border-color);
  max-width: 600px;
  background: var(--item-bg);
  overflow: hidden;
	transition: background 0.2s;
	border: none;
	z-index: 999;
}

.command-palette li {
  padding: 1rem;
  border-top: 1px solid var(--item-border);
  border-bottom: 1px solid var(--item-border);
  cursor: pointer;
  transition: background 0.2s;
}

@keyframes slidedown {
  from { opacity: 0; }
  to { opacity: 1; }
}

.command-palette.loaded {
  animation: slidedown 0.2s;
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

.footer {
	position: absolute;
	background-color: var(--item-bg);
  bottom: 0;
	left: 0;
	width: 100vw;
  margin-left: calc(50% - 50vw);
}

</style>

<main><div class="input-container">
  <input type="text" style="font-family: menlo;" bind:value={input} placeholder="Search for a snippet..." />
</div>
{#if results.length}
  <ul class="command-palette" class:loaded={hasResults}>
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

<div class="footer">
<p style="text-align: left; margin-left: 1rem;"> Made with by <a href="https://github.com/barrettjflowers">Barrett Flowers</a>. </p>
</div>
