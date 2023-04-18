<script>
  import { onMount } from "svelte";
  import { dataset_dev } from "svelte/internal";
  import { get } from "svelte/store";

  let categories = [];
  let chosenDifficulty;
  let chosenCategory;
  let questions;
  let qNumber = 0;
  let answers = [];
  let score = 0;

  onMount(async () => {
    try {
      const result = await fetch("https://opentdb.com/api_category.php");
      const data = await result.json();
      categories = data.trivia_categories;
    } catch (err) {
      console.log(err);
    }
  });

  const startGame = async (e) => {
    chosenDifficulty = e.target[0].value;
    chosenCategory = e.target[1].value;

    try {
      const result = await fetch(
        `https://opentdb.com/api.php?amount=10&category=${chosenCategory}&difficulty=${chosenDifficulty}&type=multiple`
      );
      const data = await result.json();
      questions = data.results;
    } catch (err) {
      console.log(err);
    }

    getAnswers(questions[qNumber]);
  };

  const getAnswers = (question) => {
    answers = question.incorrect_answers;
    const randomIndex = Math.floor(Math.random() * (answers.length + 1));

    answers.splice(randomIndex, 0, question.correct_answer);
  };

  const checkAnswer = (answer) => {
    if (answer === questions[qNumber].correct_answer) {
      console.log("correct!");
      score++;
    } else {
      console.log("incorrect...");
    }
    qNumber++;
    if (qNumber < 10) {
      getAnswers(questions[qNumber]);
    }
  };

  function htmlUnescape(str) {
    return str
        .replaceAll('&quot;', '"')
        .replaceAll('&#039;', "'")  
        .replaceAll('&rsquo;', "'") 
        .replaceAll('&amp;', "'")
        .replaceAll('&oacute;', "ó")
        .replaceAll('&Oacute;', "Ó")
        .replaceAll('&hellip;', '…')
        .replaceAll('&ldquo;', '"')
        .replaceAll('&rdquo;', '"')
        .replaceAll('&Eacute;', 'É')
        .replaceAll('&eacute;', 'é')
}
</script>

<main>
  <h1>Trivia!</h1>
  {#if !questions}
    <form on:submit|preventDefault={startGame}>
      <label for="difficulty">Choose difficulty:</label>
      <select id="difficulty">
        <option value="easy">Easy</option>
        <option value="medium">Medium</option>
        <option value="hard">Hard</option>
      </select>

      <label for="category">Choose a category:</label>
      <select id="category">
        {#each categories as category}
          <option value={category.id}>{category.name}</option>
        {/each}
      </select>

      <button type="submit">Let's Play!</button>
    </form>
  {:else}
    <div>
      {#if qNumber < 10}
        <h2>{htmlUnescape(questions[qNumber].question)}</h2>
        <div>
          {#each answers as answer}
            <button on:click={() => checkAnswer(answer)}>{htmlUnescape(answer)}</button>
          {/each}
        </div>
        <p>Question {qNumber + 1} of 10</p>
      {:else}
        <h2>Game over</h2>
        <p>Your score: {score}/10</p>
      {/if}
    </div>
  {/if}
</main>
