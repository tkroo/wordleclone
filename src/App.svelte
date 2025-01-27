<script lang="ts">
  import PlayButton from './lib/PlayButton.svelte';
  import {allWords, answers} from './lib/wordlewordlist.js';
  import confetti from 'canvas-confetti';
  const title = 'wordle clone';
  let word = $state('');
  const abc = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'];
  const alphabet = abc.concat(abc.map(x => x.toLowerCase()));
  let qwerty = [['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'], ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'], ['ENTER', 'Z', 'X', 'C', 'V', 'B', 'N', 'M' ,'DELETE']];
  let qwerty2 = $state(qwerty.map(x => x.map(y => ({'value':y, 'status': ''}))));
  let solved = $state(false);
  let message = $state(' ');
  let guess:string = $state('');
  let board2 = $state([ { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) } ]);
  let currentAttempt = $state(0);
  let debug = $state(false);
  let gameState = $state(true);
  let unknownWord = $state(false);
  // let playbutton;
  function chooseWord() {
    return answers[Math.floor(Math.random() * answers.length)].toLowerCase();
  }

  function setup() {
    word = chooseWord();
    solved = false;
    gameState = true;
    message = ' ';
    guess = '';
    board2 = [ { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) }, { word:'     ', matches: new Array(5) } ];
    qwerty2 = qwerty.map(x => x.map(y => ({'value':y, 'status': ''})));
    currentAttempt = 0;
  }

  setup();

  function handleKeyDown(e: KeyboardEvent) {
    if (e.key === '-') {
      debug = !debug;
    }
    if (e.key === '=') {
      setup();
    }
    if(gameState) {
      if (alphabet.includes(e.key)) {      
        let k = e.key;
        letterPressed(k);
      }
      if (e.key === 'Enter') {
        unknownWord = false;
        e.preventDefault();
        enterPressed();
      }
      if (e.key === 'Backspace' || e.key === 'Delete') {
        unknownWord = false;
        deletePressed();
      }
    }
  }

  function letterPressed(k: string) {
    if(guess.length < 5) {
      let w = guess + k;
      guess = w;
      board2[currentAttempt].word = guess.padEnd(5, ' ');
    }
  }

  function enterPressed() {
    if(guess.length == 5) {
      unknownWord = false;
      compareGuess();
    }
  }

  function deletePressed() {
    let w = guess.slice(0, -1);
    guess = w;
    board2[currentAttempt].word = guess.padEnd(5, ' ');
  }

  function compareGuess() {
    if(!allWords.includes(guess.toLowerCase())) {
      message = `${guess.toUpperCase()} is an unknown word!`;
      unknownWord = true;
      return;
    }
    
    for (let i = 0; i < 5; i++) {
      const letter = guess[i].toLowerCase();
      if (word.includes(letter)) {
        board2[currentAttempt].matches[i] = 'i';
        updateQwerty(letter, 'i');
      } else {
        board2[currentAttempt].matches[i] = 'x';
        updateQwerty(letter, 'x');
      }
      if (letter == word[i]) {
        board2[currentAttempt].matches[i] = 'c';
        updateQwerty(letter, 'c');
      }
    }
    if(word == guess.toLowerCase()) {
      solved = true;
      gameState = false;
      message = currentAttempt == 0 ? 'You got it on the first try!' : `You got it in ${currentAttempt + 1} tries!`;
      celebrate();
      return;
    }
    if(currentAttempt == 5) {
      message = `Rats! The word was ${word.toUpperCase()}`;
      gameState = false;
      solved = false;
      return;
    }
    guess = '';
    message = '';
    currentAttempt = currentAttempt + 1;
    unknownWord = false;
  }
  
  function updateQwerty(letter: string, status: string) {
    for(let i = 0; i < 3; i++) {
      let match = qwerty2[i].find(x => x.value.toLowerCase() == letter.toLowerCase());
      if(match != undefined && match.status != 'c') {
        match.status = status;
      }
    }
  }

  function celebrate() {
    var defaults = {
      spread: 360,
      ticks: 50,
      gravity: 0,
      decay: 0.94,
      startVelocity: 30,
      colors: ['001a6e', '074799', '009990', 'e1ffbb', 'ffcc00']
    };

    function shoot() {
      confetti({
        ...defaults,
        particleCount: 40,
        scalar: 1.2,
        shapes: ['star']
      });

      confetti({
        ...defaults,
        particleCount: 10,
        scalar: 5,
        shapes: ['star']
      });
    }

    setTimeout(shoot, 0);
    setTimeout(shoot, 200);
  }

</script>

<svelte:body on:keydown={handleKeyDown} />
<main>
  <div class="titlewrap">
    {#each title.split('') as letter}
      <div class="titleletter">{letter}</div>
    {/each}
  </div>
  {#if debug}
  <h3 class="test">{word}</h3>
  <!-- guess: {guess}  | guess.length: {guess.length} | currentAttempt: {currentAttempt} -->
  {/if}
  <div class="wordgrid" class:solved={solved}>
    {#each board2 as g, row}
      {#each g.word as letter, i}
        <div
          class="letter"
          class:entered={letter != ' '}
          class:bounce={g.matches[i] == 'c' && row == currentAttempt}
          class:unknownWord={unknownWord && row == currentAttempt}
          data-status={g.matches[i]}
          data-row={row}
          style="animation-delay: {i * 50}ms">{letter}
        </div>
      {/each}
    {/each}
  </div>
  <div class="message">{message} {#if !gameState}<PlayButton {setup} />{/if}</div>
  <div class="qwertygrid">
    {#each qwerty2 as row}
      <div class="row">
      {#each row as letter}
        {#if letter.value == 'ENTER'}
          <button onclick={() => enterPressed()} class="qwertyletter" data-status={letter.status} data-val={letter.value} aria-label="Enter">{letter.value}</button>
        {:else if letter.value == 'DELETE'}
          <button onclick={() => deletePressed()} class="qwertyletter" data-status={letter.status} data-val={letter.value} aria-label="Delete">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M11 10L15 14M11 14L15 10M2.7716 13.5185L7.43827 17.5185C7.80075 17.8292 8.26243 18 8.73985 18H18C19.1046 18 20 17.1046 20 16V8C20 6.89543 19.1046 6 18 6H8.73985C8.26243 6 7.80075 6.17078 7.43827 6.48149L2.7716 10.4815C1.84038 11.2797 1.84038 12.7203 2.7716 13.5185Z" stroke="#f6f5f4" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> </g></svg>
          </button>
        {:else}
          <button onclick={() => letterPressed(letter.value)} class="qwertyletter" data-status={letter.status} data-val={letter.value}>{letter.value}</button>
        {/if}
      {/each}
      </div>
    {/each}
  </div>  
</main>

<style>
  .test {
    text-transform: uppercase;
    letter-spacing: normal;
    color: #444;
  }

  .titlewrap {
    --lr: 2rem;
    margin: 0 var(--lr) 1.5rem var(--lr);
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }

  .titleletter {
    --wh: 1rem;
    color: #eee;
    font-size: 2rem;
    font-weight: 400;
    text-transform: uppercase;
    display: flex;
    justify-content: center;
    align-items: center;
    color:hsl(202, 100%, 100%);
  }


  .wordgrid {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 0.5rem;
    width: fit-content;
    margin: 0 auto 2rem auto;
  }

  .message {
    margin-bottom: 2rem;
    height: 2rem;
    font-size: 1.2rem;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .letter {
    --s: 4rem;
    width: var(--s);
    height: var(--s);
    border: 2px solid hsl(0, 0%, 27%);
    font-size: 3rem;
    font-weight: bold;
    text-transform: uppercase;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
		transform-style: preserve-3d;
  }

  .letter.entered {
    border-color: hsl(0, 0%, 47%);
  }

  .qwertygrid {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
  }

  .qwertygrid .row {
    display: flex;
    flex-direction: row;
    gap: 0.75rem;
    margin-bottom: 0.5rem;
  }

  .qwertyletter {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0.5rem 1rem;
    width: 0.6rem;
    height: 3rem;
    background-color: #818384;
    color: #f8f8f8;
    font-size: 1rem;
    font-weight: bold;
    border-radius: 0.25rem;
  }

  .solved .bounce {
    animation: bounce 0.7s;
  }

  .unknownWord.letter {
    animation: shake 0.3s;
  }

  .qwertyletter[data-val='ENTER'], .qwertyletter[data-val='DELETE'] {
    width: 4rem;
    font-size: 0.8rem;
  }

  .letter[data-status='c'], .qwertyletter[data-status='c'] {
    background-color: #538d4e;
    border-color: #538d4e;
  }
  .letter[data-status='i'], .qwertyletter[data-status='i'] {
    background-color: #b59f3b;
    border-color: #b59f3b;
  }
  .letter[data-status='x'], .qwertyletter[data-status='x'] {
    background-color: #3a3a3c;
    border-color: #3a3a3c;
  }

  @keyframes bounce {
		0%,
		20% {
			transform: translateY(0);
		}
		40% {
			transform: translateY(-20px);
		}
		50% {
			transform: translateY(5px);
		}
		60% {
			transform: translateY(-15px);
		}
		80% {
			transform: translateY(2px);
		}
		100% {
			transform: translateY(0);
		}
	}

  @keyframes shake {
  0% {
    transform: translate(1px);
  }
  20% {
    transform: translate(-2px);
  }
  40% {
    transform: translate(2px);
  }
  60% {
    transform: translate(-2px);
  }
  80% {
    transform: translate(2px);
  }
  100% {
    transform: translate(1px);
  }
}
  @keyframes shake2 {
  0% {
    transform: translate(1px);
  }
  10% {
    transform: translate(-2px);
  }
  20% {
    transform: translate(2px);
  }
  30% {
    transform: translate(-2px);
  }
  40% {
    transform: translate(2px);
  }
  50% {
    transform: translate(-2px);
  }
  60% {
    transform: translate(2px);
  }
  70% {
    transform: translate(-2px);
  }
  80% {
    transform: translate(2px);
  }
  90% {
    transform: translate(-2px);
  }
  100% {
    transform: translate(1px);
  }
}


</style>
