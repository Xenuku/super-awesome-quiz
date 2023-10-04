<script>
	import { onMount } from 'svelte';
	import Icon from '$lib/icon.svelte';

	const quizdata = 'https://super-awesome-quiz.vercel.app/questions';
	let quizzes = null;
	let randomQuiz = {};
	let error = false;
	let score = 0;
	let streak = 0;
	let correct = null;

	// On component mount, retrieve quiz data
	onMount(async function () {
		try {
			const res = await fetch(quizdata);
			if (!res.ok) {
				throw new Error(`The super awesome quiz API is too popular, please try again!`);
			}
			quizzes = await res.json();
			randomQuiz = getNewQuiz();
		} catch (err) {
			console.error(err);
			error = true;
		}
	});

	/**
	 * Generate a number from 0 - the max items in an array
	 * @param length - Array length/max number
	 */
	function generateNumber(length) {
		return Math.floor(Math.random() * length);
	}

	/**
	 * Get a new random quiz from the API
	 * Construct a quiz object from the results to use within the application
	 */
	function getNewQuiz() {
		// Clear error message when getting a new quiz
		error = false;
		const newQuizNumber = generateNumber(quizzes.sections.length);
		const category = quizzes.sections[newQuizNumber];
		const newQuestionNumber = generateNumber(category.questions.length);
		const question = category.questions[newQuestionNumber];

		const quiz = {
			category: category.section_name,
			question: question.question_text,
			id: question.id,
			type: question.answer_type,
			answers: question.answer_options
		};
		return quiz;
	}

	async function submitAnswer(questionId, answerId) {
		const answers = { answers: [{ id: questionId, answer: answerId }] };
		// Clear any error messages
		error = false;
		try {
			const res = await fetch('https://super-awesome-quiz.vercel.app/score', {
				method: 'post',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(answers)
			});
			if (!res.ok) {
				throw new Error(`The super awesome quiz API is too popular, please try again!`);
			}
			const result = await res.json();
			// Set whether the answer is correct or not for visual aid
			result.score === 0 ? (correct = false) : (correct = true);

			// Timeout of one second to show a thumbs up or down before moving on
			setTimeout(() => {
				score += result.score;
				// Reset the streak if the score is 0 (wrong answer) or increase by 1
				result.score === 0 ? (streak = 0) : streak++;
				// Get a new quiz
				randomQuiz = getNewQuiz();
				// Set correct to null to remove the icon (thumbs up or down)
				correct = null;
			}, 1000);
		} catch (err) {
			console.error(err);
			error = true;
		}
	}
</script>

<svelte:head>
	<title>Super Awesome Quizzes</title>
</svelte:head>

<div class="container">
	{#if error}
		<h2 class="error">The super awesome quiz API is too popular! Please try your request again</h2>
	{/if}
	{#if quizzes && randomQuiz}
		<div class="scoring">
			Score <span class="score">{score}</span> - Streak <span class="score">{streak}</span>
		</div>
		<div 
			class="quiz-holder" 
			class:science={randomQuiz.category == 'Science'}
			class:music={randomQuiz.category == 'Music'}
			class:movies={randomQuiz.category == 'Movies'}
			class:geography={randomQuiz.category == 'Geography'}
			class:food={randomQuiz.category == 'Food'}
			>
			<div class="quiz-header">
				<h1>
					{randomQuiz.category}
					<Icon type={randomQuiz.category} />
				</h1>
			</div>
			{#if correct != null}
				<div class="response-holder">
					<img
						src="/images/{correct ? 'correct' : 'incorrect'}.png"
						alt="The answer was {correct ? 'correct' : 'incorrect'}"
					/>
				</div>
			{/if}
			<div class="questions">
				<strong>{randomQuiz.question}</strong>
				<div>
					{#if randomQuiz.answers}
						<ul>
							{#each randomQuiz.answers as answer}
								<li>
									<button on:click={submitAnswer(randomQuiz.id, answer.id)}>{answer.text}</button>
								</li>
							{/each}
						</ul>
					{/if}
				</div>
			</div>
		</div>
	{:else}
		<h2>Loading..</h2>
		<span data-sveltekit-reload>Too popular? <a href="/">Refresh</a> the page</span>
	{/if}
</div>

<style>
	* {
		font-family: Verdana, Geneva, Tahoma, sans-serif;
	}
	.container {
		margin: 0 auto;
		width: 80vw;
		min-height: 73vh;
		background: #1d1d1d;
		padding: 20px;
		text-align: center;
		color: #e3e3e3;
	}
	.quiz-holder {
		min-width: 40vw;
		width: 40vw;
		margin: 0 auto;
		border-radius: 2.5%;
	}
	.questions, .response-holder {
		padding-top: 1rem;
	}
	.questions {
		padding: 2.5%;
	}
	.score {
		color: aqua;
	}
	.scoring {
		font-size: 1.3rem;
		margin-bottom: 20px;
	}
	.error {
		color: rgb(163, 3, 3);
	}
	h1 {
		font-size: 2rem;
		margin: 0;
		padding: 2% 0;
	}
	ul {
		list-style: none;
		line-height: 1.5;
		margin-left: -40px;
	}
	a {
		text-decoration: none;
	}
	strong {
		font-size: 1.4rem;
		text-align: justify;
	}
	button {
		font-size: 1.5rem;
		padding: 10px;
		width: 30vw;
		cursor: pointer;
		margin-bottom: 20px;
		border: 0;
		color: #e3e3e3;
	}
	
	/* Conditional styles depending which category is chosen */
	.movies {
		background: #33C4C2;
	}
	.movies button {
		background: #187876;
	}
	.movies h1 {
		border-bottom: 2px solid #187876;
	}
	.science {
		background: #FF858B;
	}
	.science button {
		background: #b24751;
	}
	.science h1 {
		border-bottom: 2px solid #b24751;
	}
	.music {
		background: #957FF9;
	}
	.music button {
		background: #7E63F8;
	}
	.music h1 {
		border-bottom: 2px solid #7E63F8;
	}
	.food {
		background: #4CB732;
	}
	.food button {
		background: #368023;
	}
	.food h1 {
		border-bottom: 2px solid #368023;
	}
	.geography {
		background: #53b8fe;
	}
	.geography button {
		background: #367db1;
	}
	.geography h1 {
		border-bottom: 2px solid #367db1;
	}

	button:hover {
		background: #25201d;
		color: orange;
	}

	@media(max-width: 1200px) {
		.quiz-holder {
			min-width: 80vw;
		}
		button {
			width: 60vw;
		}
	}
</style>
