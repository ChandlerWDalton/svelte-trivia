<script>
	import { onMount } from "svelte";
    import { each } from "svelte/internal";
	let categories = [];
	let quiz = [];
	let quizStarted = false;
	let step = 0;
	let points = 0;

	onMount(async () => {
		categories = await getCategories();
	});

	let options = {
		difficulty: '',
		category: ''
	}

	let difficulties = ['easy', 'medium', 'hard'];

	async function getCategories(){
		try{
			const response = await fetch(`https://opentdb.com/api_category.php`);
			const categories = await response.json();
			return categories.trivia_categories;
		} catch(err){
			console.error('Error getting categories', err)
		}
	}

	async function getQuiz(){
		try{

			const response = await fetch(`https://opentdb.com/api.php?amount=10&type=multiple&category=${options.category}&difficulty=${options.difficulty}`);
			const newQuiz = await response.json();
			for (const question of newQuiz.results){
				let answers = [];
				question.choices = [];
				for (const incorrect of question.incorrect_answers){
					answers.push({
						answer: incorrect,
						pass: false
					})
				}
				answers.push({
					answer: question.correct_answer,
					pass: true

				})
				question.choices = shuffle(answers);
				question.question = question.question.replaceAll('&quot;', '"')
				question.question = question.question.replaceAll('&#039;', `'`)
			}
			quiz = newQuiz.results;
			console.log(quiz);
			quizStarted = true;
		} catch(err){
			console.error('Error getting categories', err)
		}
	}

	function shuffle(array) {
		let currentIndex = array.length,  randomIndex;

		// While there remain elements to shuffle.
		while (currentIndex != 0) {

			// Pick a remaining element.
			randomIndex = Math.floor(Math.random() * currentIndex);
			currentIndex--;

			// And swap it with the current element.
			[array[currentIndex], array[randomIndex]] = [
			array[randomIndex], array[currentIndex]];
		}

		return array;
	}

	function answerQuestion(result){
		if(result){
			points++
		}
		step++
	}

	function playAgain(){
		quizStarted = false;
		step = 0;
		points = 0;
	}

</script>

<main>
	<h1>Welcome to Trivia!</h1>
	{#if !quizStarted}
	<h4>Quiz Options</h4>
	<div>
		<select bind:value={options.difficulty}>
			{#each difficulties as difficulty}
				<option value={difficulty}>{difficulty}</option> 
			{/each}
		</select>
		<select bind:value={options.category}>
			{#each categories as category}
				<option value={category.id}>{category.name}</option> 
			{/each}
		</select>
	</div>
	<button on:click={getQuiz}>Let's Go!</button>
	{:else if step != 10}
	<div>
		<h6>{points}/10</h6>
		<h4>Question {step + 1}</h4>
		<h4>{quiz[step].question}</h4>
		{#each quiz[step].choices as choice}
			<button on:click={answerQuestion(choice.pass)}>{choice.answer}</button>
		{/each}
	</div>
	{:else}
	<h2>Final Score: {points}/10</h2>
	<button on:click={playAgain}>Play Again</button>
	{/if}

</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		margin: 0 auto;
	}

	h1 {
		color: black;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	button {
		margin: .5em;
		border-radius: 10px;
		background-color: rgb(80, 173, 255);
		padding: .5em;
	}
</style>