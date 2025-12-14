<div class="header">
<span class="title-inner">
<h1 class= "title" style="text-align: center; margin-bottom: 1.5rem; margin-top: 0rem;">book<span class="accent">zf</span>
</h1>
<svg class="logo" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 18.291 14.3262">
 <g>
  <rect height="14.3262" opacity="0" width="18.291" x="0" y="0"/>
  <path d="M4.33594 0C2.62695 0 0.878906 0.810547 0.0878906 2.1875C0 2.34375 0 2.45117 0 2.76367L0 13.6719C0 14.0625 0.253906 14.3066 0.683594 14.3066C0.878906 14.3066 1.07422 14.248 1.24023 14.1016C2.20703 13.291 3.45703 12.8613 4.69727 12.8613C5.83008 12.8613 6.94336 13.2227 7.79297 13.9551C7.88086 14.0332 7.98828 14.0723 8.07617 14.0723C8.25195 14.0723 8.39844 13.9355 8.39844 13.7305L8.39844 2.25586C8.39844 2.04102 8.38867 2.01172 8.22266 1.77734C7.50977 0.683594 6.00586 0 4.33594 0ZM13.5938 0C11.9238 0 10.4199 0.683594 9.70703 1.77734C9.54102 2.01172 9.53125 2.04102 9.53125 2.25586L9.53125 13.7305C9.53125 13.9355 9.67773 14.0723 9.85352 14.0723C9.94141 14.0723 10.0488 14.0332 10.1367 13.9551C10.9863 13.2227 12.0996 12.8613 13.2227 12.8613C14.4727 12.8613 15.7227 13.291 16.6895 14.1016C16.8555 14.248 17.0508 14.3066 17.2461 14.3066C17.6758 14.3066 17.9297 14.0625 17.9297 13.6719L17.9297 2.76367C17.9297 2.45117 17.9297 2.34375 17.8418 2.1875C17.0508 0.810547 15.3027 0 13.5938 0Z" fill="white" fill-opacity="0.85"/>
 </g>
</svg>
</span>
<hr style="margin-top: 0rem; margin-bottom: 1rem; border: none; border-top: 1px solid var(--text-color);">
<p style="margin: 0.03rem 0; text-align: center;"> Do you switch between audiobooks and physical books? Bookzf can help you find the page you're looking for. </p>
<p style="text-align: center;"> A fuzzy finder for book pages. </p>
</div>

<script>
   import '../app.css';
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
</div>
