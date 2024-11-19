<script lang="ts">
	import { onMount } from 'svelte';
	import WordCloud from './Wordcloud.svelte';

	let words: { text: string; count: number }[] = [];

	const wordCounts = new Map<string, number>();

	let posts = 0;

	let countdown = 3;

	let firstCountdown = true;

	let width: number;
	let height: number;

	const fillerWords = new Set([
		'a',
		'an',
		'the',
		'and',
		'or',
		'but',
		'if',
		'then',
		'so',
		'because',
		'as',
		'of',
		'at',
		'by',
		'for',
		'with',
		'about',
		'against',
		'between',
		'into',
		'through',
		'during',
		'before',
		'after',
		'above',
		'below',
		'to',
		'from',
		'up',
		'down',
		'in',
		'out',
		'on',
		'off',
		'over',
		'under',
		'again',
		'further',
		'then',
		'once',
		'here',
		'there',
		'when',
		'where',
		'why',
		'how',
		'all',
		'any',
		'both',
		'each',
		'few',
		'more',
		'most',
		'other',
		'some',
		'such',
		'no',
		'nor',
		'not',
		'only',
		'own',
		'same',
		'so',
		'than',
		'too',
		'very',
		's',
		't',
		'can',
		'will',
		'just',
		'don',
		'should',
		'now',
		'i',
		'me',
		'my',
		'myself',
		'we',
		'our',
		'ours',
		'ourselves',
		'you',
		'your',
		'yours',
		'yourself',
		'yourselves',
		'he',
		'him',
		'his',
		'himself',
		'she',
		'her',
		'hers',
		'herself',
		'it',
		'its',
		'itself',
		'they',
		'them',
		'their',
		'theirs',
		'themselves',
		'what',
		'which',
		'who',
		'whom',
		'this',
		'that',
		'these',
		'those',
		'am',
		'is',
		'are',
		'was',
		'were',
		'be',
		'been',
		'being',
		'have',
		'has',
		'had',
		'having',
		'do',
		'does',
		'did',
		'doing',
		'a',
		'an',
		'the',
		'and',
		'or',
		'but',
		'if',
		'then',
		'else',
		'because',
		'as',
		'of',
		'at',
		'by',
		'for',
		'with',
		'about',
		'against',
		'between',
		'into',
		'through',
		'during',
		'before',
		'after',
		'above',
		'below',
		'to',
		'from',
		'up',
		'down',
		'in',
		'out',
		'on',
		'off',
		'over',
		'under',
		'again',
		'further',
		'then',
		'once',
		'here',
		'there',
		'when',
		'where',
		'why',
		'how',
		'all',
		'any',
		'both',
		'each',
		'few',
		'more',
		'most',
		'other',
		'some',
		'such',
		'no',
		'nor',
		'not',
		'only',
		'own',
		'same',
		'so',
		'than',
		'too',
		'very',
		's',
		't',
		'can',
		'will',
		'just',
		'don',
		'should',
		'now'
	]);

	onMount(() => {
		function updateDimensions() {
			width = window.innerWidth;
			height = window.innerHeight;
		}

		updateDimensions();

		setTimeout(() => {
			updateWords();
			setInterval(updateWords, 10000);
		}, 3000);

		let countdownInterval = setInterval(() => {
			countdown--;

			if (countdown === 0) {
				firstCountdown = false;
				countdown = 10;
			}
		}, 1000);

		window.addEventListener('resize', updateDimensions);

		function secondsToMicroseconds(seconds: number) {
			return Math.floor(seconds * 1000000);
		}

		const startTime = Date.now() * 1000 - secondsToMicroseconds(10);
		console.log(startTime);

		// WebSocket URL from BlueSky
		const url =
			'wss://jetstream2.us-east.bsky.network/subscribe?wantedCollections=app.bsky.feed.post&cursor=' +
			startTime;

		// WebSocket logic
		const ws = new WebSocket(url);
		ws.onopen = () => {
			console.log('Connected to BlueSky WebSocket');
		};

		ws.onmessage = (event) => {
			const json = JSON.parse(event.data);

			if (
				json.kind === 'commit' &&
				json.commit.collection === 'app.bsky.feed.post' &&
				json.commit.operation === 'create' &&
				json.commit.record.text &&
				(!json.commit.record.langs || json.commit.record.langs.includes('en'))
			) {
				// get text of post
				const text: string = json.commit.record.text;

				// split text into words (split by whitespace, newlines, punctuation)
				const words = text.split(/[\s\n\.,!?;]+/);

				words.forEach((word) => {
					// skip words that are empty or not alphanumeric
					if (!word || !word.match(/^[a-zA-Z]+$/) || fillerWords.has(word.toLowerCase())) {
						return;
					}
					// turn to lowercase
					word = word.toLowerCase();

					wordCounts.set(word, (wordCounts.get(word) || 0) + 1);
				});

				posts++;
			}
		};

		return () => {
			ws.close();
			window.removeEventListener('resize', updateDimensions);

			clearInterval(countdownInterval);
		};
	});

	function updateWords() {
		// get 200 most common words
		const mostCommonWords = Array.from(wordCounts.entries())
			.sort((a, b) => b[1] - a[1])
			.slice(0, 300);

		words = mostCommonWords.map(([text, count]) => ({ text, count }));

		// clear wordCounts
		wordCounts.clear();
	}
</script>

<div class="flex h-screen w-screen items-center justify-center overflow-hidden">
	{#if width && height}
		<WordCloud {words} {width} {height} />
	{/if}
</div>

{#if countdown > 0 && firstCountdown}
	<div class="absolute inset-0 flex h-full w-full items-center justify-center text-8xl font-bold">
		{countdown}
	</div>
{/if}

<div
	class="absolute bottom-0 left-0 right-0 flex w-full justify-center bg-white/50 px-4 py-2 text-sm font-semibold text-black backdrop-blur-sm"
>
	<div>
		{#if countdown > 0 && firstCountdown}
			collecting posts and counting words...
		{:else}
			showing most common words of the last 10 seconds excluding filler words, update in {countdown}...
		{/if}
	</div>
</div>

<div
	class="absolute left-0 top-0 flex w-full items-center justify-between px-4 py-2 text-sm font-semibold"
>
	<div>
		by <a
			href="https://flo-bit.dev/"
			class="transition-colors duration-100 hover:text-cyan-600"
			target="_blank">flo-bit</a
		>
	</div>

	<div>
		<a
			class="z-10 text-black transition-colors duration-100 hover:text-cyan-600"
			href="https://github.com/flo-bit/bluesky-wordcloud"
			target="_blank"
		>
			<span class="sr-only">GitHub Code</span>
			<svg fill="currentColor" viewBox="0 0 24 24" aria-hidden="true" class="size-6">
				<path
					fill-rule="evenodd"
					d="M12 2C6.477 2 2 6.484 2 12.017c0 4.425 2.865 8.18 6.839 9.504.5.092.682-.217.682-.483 0-.237-.008-.868-.013-1.703-2.782.605-3.369-1.343-3.369-1.343-.454-1.158-1.11-1.466-1.11-1.466-.908-.62.069-.608.069-.608 1.003.07 1.531 1.032 1.531 1.032.892 1.53 2.341 1.088 2.91.832.092-.647.35-1.088.636-1.338-2.22-.253-4.555-1.113-4.555-4.951 0-1.093.39-1.988 1.029-2.688-.103-.253-.446-1.272.098-2.65 0 0 .84-.27 2.75 1.026A9.564 9.564 0 0112 6.844c.85.004 1.705.115 2.504.337 1.909-1.296 2.747-1.027 2.747-1.027.546 1.379.202 2.398.1 2.651.64.7 1.028 1.595 1.028 2.688 0 3.848-2.339 4.695-4.566 4.943.359.309.678.92.678 1.855 0 1.338-.012 2.419-.012 2.747 0 .268.18.58.688.482A10.019 10.019 0 0022 12.017C22 6.484 17.522 2 12 2z"
					clip-rule="evenodd"
				/>
			</svg>
		</a>
	</div>
</div>

<style>
	div {
		overflow: hidden;
	}
</style>
