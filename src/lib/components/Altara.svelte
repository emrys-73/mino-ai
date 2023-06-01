<script lang="ts">
	import ChatMessage from '$lib/components/ChatMessage.svelte'
	import type { ChatCompletionRequestMessage } from 'openai'
	import { SSE } from 'sse.js'

	let query: string = ''
	let answer: string = ''
	let loading: boolean = false
	let chatMessages: ChatCompletionRequestMessage[] = []
	let scrollToDiv: HTMLDivElement

	function scrollToBottom() {
		setTimeout(function () {
			scrollToDiv.scrollIntoView({ behavior: 'smooth', block: 'end', inline: 'end' })
		}, 100)
	}

	const MESSAGE_LIMIT = 30;
	let messageCount = 0;

	const handleSubmit = async () => {
		if (messageCount >= MESSAGE_LIMIT - 1) {
			return;
		}

		messageCount++;

		loading = true
		chatMessages = [...chatMessages, { role: 'user', content: query }]

		const controller = new AbortController();
		const { signal } = controller;
		const timeout = setTimeout(() => {
			controller.abort();
		}, 20000); // Set timeout to 20 seconds (20000 milliseconds)

		try {
			const eventSource = new SSE('/api/chat', {
				headers: {
					'Content-Type': 'application/json'
				},
				payload: JSON.stringify({ messages: chatMessages }),
				// signal // Pass the signal to the SSE constructor
			})

			query = ''

			eventSource.addEventListener('error', handleError)

			eventSource.addEventListener('message', (e) => {
				scrollToBottom();
				try {
					loading = false
					if (e.data === '[DONE]') {
						chatMessages = [...chatMessages, { role: 'assistant', content: answer }]
						answer = ''
						return
					}

					const completionResponse = JSON.parse(e.data)
					const [{ delta }] = completionResponse.choices

					if (delta.content) {
						answer = (answer ?? '') + delta.content
					}
				} catch (err: any) {
					if (err.message === 'Timeout') {
						chatMessages = [...chatMessages, { role: 'assistant', content: 'Sorry, we\'re at capacity right now. Please try again later.' }]
					} else {
						chatMessages = [...chatMessages, { role: 'assistant', content: 'Sorry, we ran into a problem. Please try again later.' }]
						handleError(err)
					}
				}
			})

			eventSource.addEventListener('httpError', (e: any) => {
				if (e.data.status === 500) {
					console.log('Sorry, we had an error');
				}
			})

			eventSource.stream()
			scrollToBottom()
		} catch (err) {
			handleError(err);
		} finally {
			clearTimeout(timeout); // Clear the timeout if the request completes within the timeout duration
		}
	}

	function handleError<T>(err: T) {
		loading = false
		query = ''
		answer = 'Sorry, I am having too many messsages right now. Please try again later'

		console.error(err)
	}
</script>


<div class="relative">
    <div class="card">
		
        <div class="h-[600px] bg-zinc-900  bg-opacity-20 rounded-xl p-4 overflow-y-auto flex flex-col gap-4 min-w-[800px]">
            <div class="flex flex-col gap-2 text-left ">
				
                <ChatMessage type="assistant" message="Hi, what can I do for you today?" />
                {#each chatMessages as message}
                    <ChatMessage type={message.role} message={message.content} />
                {/each}
                {#if answer}
                    <ChatMessage type="assistant" message={answer} />
                {/if}
                {#if loading}
                    <div class="animate-pulse">
                        <ChatMessage type="assistant" message="Thinking..." />
                    </div>
                {/if}
				<div class=" bg-black max-w-none" bind:this={scrollToDiv} />
            </div>
            
        </div>
    </div>
</div>
<form
		class="flex max-w-3xl min-w-[700px] rounded-md gap-4 bg-transparent p-4"
		on:submit|preventDefault={() => handleSubmit()}
	>
		<input placeholder="Write a message" type="text" class="input input-bordered w-full min-w-[338px] bg-zinc-900 placeholder:italic" bind:value={query} />
		<button type="submit" class="btn bg-zinc-900"> ➟	 </button>
	</form>
    <!--
		<div>
        <a href="/">
            <p class="text-sm pb-6 text-gray-700">Home</p>
        </a>
    </div>

	-->