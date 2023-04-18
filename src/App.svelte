<script>
  import { onMount } from "svelte";
  import { dataset_dev } from "svelte/internal";

  let categories = [];
  let chosenDifficulty;
  let chosenCategory;
  let questions;
  let qNumber = 0;
  let answers = []

  $: console.log(questions)

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
  };

  
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
      <h2>{questions[qNumber].question}</h2>

    </div>
  {/if}
</main>
