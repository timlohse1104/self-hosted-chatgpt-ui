<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import SvelteMarkdown from 'svelte-markdown';
    import Code from './Code.svelte';
    import type { Message, Model, Usage } from './Types.svelte';

    // Marked options
    const markedownOptions = {
      gfm: true, // Use GitHub Flavored Markdown
      breaks: true, // Enable line breaks in markdown
      mangle: false // Do not mangle email addresses
    }

    export let messages : Message[]
    export let input: HTMLTextAreaElement
    export let defaultModel: Model
    let isDeleteButtonHidden = true

    const dispatch = createEventDispatcher()

    // Reference: https://openai.com/pricing#language-models
    const tokenPrice : Record<string, [number, number]> = {
      'gpt-4-32k': [0.00006, 0.00012], // $0.06 per 1000 tokens prompt, $0.12 per 1000 tokens completion
      'gpt-4': [0.00003, 0.00006], // $0.03 per 1000 tokens prompt, $0.06 per 1000 tokens completion
      'gpt-3.5': [0.000002, 0.000002] // $0.002 per 1000 tokens (both prompt and completion)
    }

    const getPrice = (tokens: Usage, model: Model): number => {
      for (const [key, [promptPrice, completionPrice]] of Object.entries(tokenPrice)) {
        if (model.startsWith(key)) {
          return ((tokens.prompt_tokens * promptPrice) + (tokens.completion_tokens * completionPrice))
        }
      }

      return 0
    }

    const dispatchDeleteEvent = (message) => dispatch('delete', message)
    const showDeleteButton = () => {
      isDeleteButtonHidden = false
    }
    const hideDeleteButton = () => {
      isDeleteButtonHidden = true
    }
</script>

{#each messages as message, i}
  {#if message.role === 'user'}
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <article
      class="message is-info user-message"
      class:has-text-right={message.content.split('\n').filter((line) => line.trim()).length === 1}
      on:mouseover={showDeleteButton}
      on:mouseout={hideDeleteButton}
    >
      <div class="message-body content" style="position: relative;">
        <a
          href={'#'}
          class="greyscale is-pulled-right ml-2 is-hidden editbutton"
          on:click={() => {
            input.value = message.content
            input.focus()
          }}
        >
          ✏️
        </a>
        <button class="delete is-medium is-danger" class:is-hidden={isDeleteButtonHidden} style="position: absolute; top: 0; right: 0;" on:click={() => dispatchDeleteEvent(i)}></button>
        <SvelteMarkdown source={message.content} options={markedownOptions} renderers={{ code: Code, html: Code }}/>
      </div>
    </article>
  {:else if message.role === 'system'}
    <!-- Dont shot system messages -->
    <!-- <article class="message is-warning user-message">
      <div class="message-body content">
        <SvelteMarkdown source={message.content} options={markedownOptions} renderers={{ code: Code, html: Code }}/>
      </div>
    </article> -->
  {:else if message.role === 'error'}
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <article class="message is-danger assistant-message" on:mouseover={showDeleteButton}
    on:mouseout={hideDeleteButton}>
      <div class="message-body content" style="position: relative;">
        <button class="delete is-medium is-danger" class:is-hidden={isDeleteButtonHidden} style="position: absolute; top: 0; right: 0;" on:click={() => dispatchDeleteEvent(i)}></button>
        <SvelteMarkdown source={message.content} options={markedownOptions} renderers={{ code: Code, html: Code }}/>
      </div>
    </article>
  {:else}
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <article class="message is-success assistant-message" on:mouseover={showDeleteButton}
    on:mouseout={hideDeleteButton}>
      <div class="message-body content" style="position: relative;">
        <button class="delete is-medium is-danger" class:is-hidden={isDeleteButtonHidden} style="position: absolute; top: 0; right: 0;" on:click={() => dispatchDeleteEvent(i)}></button>
        <SvelteMarkdown source={message.content} options={markedownOptions} renderers={{ code: Code, html: Code }}/>
        {#if message.usage}
          <p class="is-size-7">
            This message was generated on <em>{message.model || defaultModel}</em> using <span class="has-text-weight-bold">{message.usage.total_tokens}</span>
            tokens ~= <span class="has-text-weight-bold">${getPrice(message.usage, message.model || defaultModel).toFixed(6)}</span>
          </p>
        {/if}
      </div>
    </article>
  {/if}
{/each}