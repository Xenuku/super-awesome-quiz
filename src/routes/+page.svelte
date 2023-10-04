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
		// On submit, clear any errors
		error = false;
		const answers = { answers: [{ id: questionId, answer: answerId }] };

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
		<h2>The super awesome quiz API is too popular! Please try your request again</h2>
	{/if}
	{#if quizzes && randomQuiz}
		<h1>
			{randomQuiz.category} <Icon type={randomQuiz.category} />
		</h1>
        <div class="scoring"> Score <span class="score">{score}</span> - Streak <span class="score">{streak}</span></div>
        {#if correct != null}
        <div>
            <img src="/images/{correct ? 'correct' : 'incorrect'}.png" alt="The answer was {correct ? 'correct' : 'incorrect'}" />
        </div>
        {/if}

		<strong>{randomQuiz.question}</strong>
		<div>
			{#if randomQuiz.answers}
				<ul>
					{#each randomQuiz.answers as answer}
						<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
						<!-- svelte-ignore a11y-click-events-have-key-events -->
						<li>
							<button on:click={submitAnswer(randomQuiz.id, answer.id)}>{answer.text}</button>
						</li>
					{/each}
				</ul>
			{/if}
		</div>
	{:else}
		<h2>Loading..</h2>
	{/if}
</div>

<style>
    * {
        font-family: 'Courier New', Courier, monospace;
    }
	.container {
		margin: 0 auto;
		width: 80vw;
		height: 90vh;
		background: #64748b;
		padding: 20px;
        text-align: center;
	}
    .score {
        color: aqua;
    }
    .scoring {
        font-size: 1.3rem;
        margin-bottom: 20px;
    }
	h1,
	h2 {
		text-align: center;
	}
    h1 {
        font-size: 2rem;
    }
	ul {
		list-style: none;
		line-height: 1.5;
        margin-left: -40px;
	}
	strong {
		font-size: 1.4rem;
		text-align: justify;
	}
	button {
        font-size: 1.5rem;
        padding: 10px;
        width: 300px;
        cursor: pointer;
        margin-bottom: 20px;
    }
</style>
