---
{"dg-publish":true,"permalink":"/flashcard/flashcards/"}
---

```html
<div id="flashcard-container" class="flashcard-wrapper"></div>

<script>
(async function () {
  const lectureFolders = ['Lecture 1', 'Lecture 2','Lecture 3', 'Lecture 4', 'Lecture 5']; 
  const githubUser = 'DanielWu06'; 
  const repo = 'ah-46-notes-digital-garden';
  const branch = 'main';

  let allText = '';

  for (let folder of lectureFolders) {
    const apiUrl = `https://api.github.com/repos/${githubUser}/${repo}/contents/${encodeURIComponent(folder)}`;

    try {
      const res = await fetch(apiUrl);
      const files = await res.json();

      for (let file of files) {
        if (file.name.endsWith('.md')) {
          const rawUrl = `https://raw.githubusercontent.com/${githubUser}/${repo}/${branch}/${encodeURIComponent(folder)}/${encodeURIComponent(file.name)}`;
          const contentRes = await fetch(rawUrl);
          const markdown = await contentRes.text();
          allText += '\n\n' + markdown;
        }
      }
    } catch (err) {
      console.error("Error loading folder:", folder, err);
    }
  }

  const cards = parseFlashcards(allText);
  displayFlashcards(cards);

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

    for (let i = cards.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [cards[i], cards[j]] = [cards[j], cards[i]];
    }

    for (let card of cards) {
      const cardElem = document.createElement('div');
      cardElem.className = 'flashcard';

      const front = document.createElement('div');
      front.className = 'front';
      front.innerHTML = `<img src="/${card.img}" alt="Flashcard Image" />`;

      const back = document.createElement('div');
      back.className = 'back';
      back.innerHTML = `<p>${card.back.replace(/\n/g, "<br>")}</p>`;

      cardElem.appendChild(front);
      cardElem.appendChild(back);
      container.appendChild(cardElem);

      cardElem.addEventListener('click', () => {
        cardElem.classList.toggle('flipped');
      });
    }
  }
})();
</script>

<style>
.flashcard-wrapper {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-top: 2rem;
}

.flashcard {
  perspective: 1000px;
  cursor: pointer;
  position: relative;
  height: 250px;
}

.flashcard .front, .flashcard .back {
  transition: transform 0.6s;
  transform-style: preserve-3d;
  backface-visibility: hidden;
  padding: 1rem;
  border: 1px solid var(--color-border);
  border-radius: 10px;
  background: var(--background-primary);
  box-shadow: var(--shadow-s);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  position: absolute;
  width: 100%;
  height: 100%;
}

.flashcard .front {
  z-index: 2;
}

.flashcard .back {
  transform: rotateY(180deg);
}

.flashcard.flipped .front {
  transform: rotateY(180deg);
}

.flashcard.flipped .back {
  z-index: 2;
  transform: rotateY(0);
}
</style>
```