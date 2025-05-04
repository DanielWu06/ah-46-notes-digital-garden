---
{"dg-publish":true,"permalink":"/flashcard/flashcards/"}
---

## Flashcard Review

This is a dynamic flashcard set pulled from all lecture notes.

<div id="flashcard-container" class="flashcard-wrapper"></div>

<script>

async function fetchAllMarkdownNotes() {
  const lectureFolders = ['Lecture 1', 'Lecture 2'];
  const baseUrl = 'https://ah-46-notes-digital-garden.vercel.app';

  let noteUrls = [];

  for (let folder of lectureFolders) {
    const folderUrl = `${baseUrl}/${encodeURIComponent(folder)}/`;
    try {
      const res = await fetch(folderUrl);
      const html = await res.text();

      // Match .md page URLs
      const regex = /href="(\/[^"]*Lecture\s\d\s-\s[^"]+)"/g;
      let match;
      while ((match = regex.exec(html)) !== null) {
        noteUrls.push(baseUrl + match[1]);
      }
    } catch (err) {
      console.error("Error loading folder", folder, err);
    }
  }

  let allText = '';
  for (let url of noteUrls) {
    try {
      const res = await fetch(url);
      const text = await res.text();
      allText += '\n\n' + text;
    } catch (err) {
      console.error("Could not fetch note:", url, err);
    }
  }

  return allText;
}

function parseFlashcards(text) {
  const regex = /<span class="hide-in-garden">\*\*Front:\*\*<\/span>\s*!\[\[(.*?)\]\]\s*\n\?\s*\n<span class="hide-in-garden">\*\*Back:\*\*<\/span>\s*([\s\S]*?)(?=\n{2,}|$)/g;
  let match;
  let cards = [];

  while ((match = regex.exec(text)) !== null) {
    cards.push({
      img: match[1],
      back: match[2].trim()
    });
  }

  return cards;
}

function displayFlashcards(cards) {
  const container = document.getElementById('flashcard-container');
  if (!container) return;

  // Shuffle cards
  for (let i = cards.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [cards[i], cards[j]] = [cards[j], cards[i]];
  }

  for (let card of cards) {
    const cardElem = document.create