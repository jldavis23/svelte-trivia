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
  let feedback = null;
  let isAnswered = false;

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
    answers = question.incorrect_answers.map((answer) => {
      return { text: answer, isCorrect: false };
    });
    const randomIndex = Math.floor(Math.random() * (answers.length + 1));

    answers.splice(randomIndex, 0, {
      text: question.correct_answer,
      isCorrect: true,
    });
  };

  const checkAnswer = (answer) => {
    isAnswered = true
    answer.isChosen = true
    if (answer.isCorrect === true) {
      feedback = "Correct!";
      score++;
    } else {
      feedback = "Incorrect...";
    }
  };

  const nextQuestion = () => {
    isAnswered = false
    feedback = null;
    qNumber++;
    if (qNumber < 10) {
      getAnswers(questions[qNumber]);
    }
  };

  function htmlReplace(str) {
    return str
      .replaceAll("&quot;", '"')
      .replaceAll("&#039;", "'")
      .replaceAll("&rsquo;", "'")
      .replaceAll("&amp;", "'")
      .replaceAll("&oacute;", "ó")
      .replaceAll("&Oacute;", "Ó")
      .replaceAll("&hellip;", "…")
      .replaceAll("&ldquo;", '"')
      .replaceAll("&rdquo;", '"')
      .replaceAll("&Eacute;", "É")
      .replaceAll("&eacute;", "é")
      .replaceAll("&aacute;", "á")
      .replaceAll("&Aaute;", "Á")
      .replaceAll("&shy;", "");
  }
</script>

<main>
  <h1>TRIVIA</h1>
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

      <button type="submit" class="lets-play">Let's Play!</button>
    </form>
  {:else}
    <div>
      {#if qNumber < 10}
        <p>Question {qNumber + 1} of 10</p>
        <h2>{htmlReplace(questions[qNumber].question)}</h2>
        <div class="answer-buttons">
          {#each answers as answer}
            {#if isAnswered}
              <button class="isAnswered" class:btn-correct={answer.isCorrect} class:btn-incorrect={!answer.isCorrect} class:btn-chosen={answer.isChosen}>{htmlReplace(answer.text)}</button>
            {:else}
              <button on:click={() => checkAnswer(answer)}
                >{htmlReplace(answer.text)}</button
              >
            {/if}
          {/each}
        </div>
        {#if feedback}
          <p>{feedback}</p>
          <button class="next" on:click={nextQuestion}>Next</button>
        {/if}
      {:else}
        <h2>Game over</h2>
        <p>Your score: {score}/10</p>
      {/if}
    </div>
  {/if}
</main>
