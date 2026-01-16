---
layout: default
title: "Quotes"
permalink: /quotes/
---

<div class="quotes-page">
  <h1>Featured Quote</h1>
  <div id="quote-display" class="quote-display">
    <blockquote id="quote-text" class="quote"></blockquote>
    <footer id="quote-author" class="quote-author"></footer>
    <div class="quote-controls">
      <button onclick="displayRandomQuote()">Next Quote</button>
    </div>
  </div>
</div>

<style>
.quote-display {
  background: #f5f5f5;
  border-left: 4px solid #333;
  padding: 2rem;
  margin: 2rem 0;
  font-style: italic;
  max-width: 600px;
}

#quote-text {
  margin: 0 0 1rem 0;
  font-size: 1.2rem;
  line-height: 1.6;
}

#quote-author {
  display: block;
  text-align: right;
  font-style: normal;
  margin-top: 1rem;
  color: #666;
}

.quote-controls {
  margin-top: 1.5rem;
  text-align: center;
}

.quote-controls button {
  background: #333;
  color: white;
  border: none;
  padding: 0.5rem 1.5rem;
  cursor: pointer;
  border-radius: 4px;
  font-size: 1rem;
}

.quote-controls button:hover {
  background: #555;
}
</style>

<script>
const quotes = [
  {
    quote: "The only way to do great work is to love what you do.",
    author: "Steve Jobs",
    source: "Stanford Commencement Address"
  },
  {
    quote: "Code is like humor. When you have to explain it, it's bad.",
    author: "Cory House"
  },
  {
    quote: "Premature optimization is the root of all evil.",
    author: "Donald Knuth",
    source: "Computer Programming as an Art"
  },
  {
    quote: "We are like butterflies who flutter for a day and think it is forever.",
    author: "Carl Sagan"
  }
];

function displayRandomQuote() {
  const randomIndex = Math.floor(Math.random() * quotes.length);
  const quote = quotes[randomIndex];

  document.getElementById('quote-text').textContent = `"${quote.quote}"`;

  let authorText = `— ${quote.author}`;
  if (quote.source) {
    authorText += `, ${quote.source}`;
  }
  document.getElementById('quote-author').textContent = authorText;
}

// Display a random quote on page load
window.addEventListener('load', displayRandomQuote);
</script>
