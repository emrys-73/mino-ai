<script lang="ts">
	import type { ChatCompletionRequestMessageRoleEnum } from 'openai'
	export let type: ChatCompletionRequestMessageRoleEnum
	export let message: string
</script>

<div class="chat {type === 'user' ? 'chat-end' : 'chat-start'} justify-end">
	<div class="chat-image avatar">
		<div class="w-8 rounded-full">
			<img
				src="https://ui-avatars.com/api/?name={type === 'user' ? 'Me' : 'AI'}"
				alt="{type} avatar"
			/>
		</div>
	</div>
	<div class="chat-header">
		{type === 'user' ? 'Me' : 'AI'}
	</div>
	<div class="chat-bubble {type == 'user' ? 'mbx_user' : 'mbx_altara'} {type === 'user' ? 'chat-bubble-primary ' : 'chat-bubble-secondary'}">
		{message}
	</div>
</div>
