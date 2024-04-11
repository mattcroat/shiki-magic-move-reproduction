<script lang="ts">
	import { onMount } from 'svelte'
	import {
		getHighlighter,
		type BundledLanguage,
		type SpecialLanguage,
		type HighlighterCore,
	} from 'shiki'
	import { codeToKeyedTokens, createMagicMoveMachine } from 'shiki-magic-move/core'
	import { MagicMoveRenderer } from 'shiki-magic-move/renderer'
	import type { MagicMoveDifferOptions, MagicMoveRenderOptions } from 'shiki-magic-move/types'

	let code: string = 'let bool;'
	let lang: BundledLanguage | SpecialLanguage = 'ts'
	let theme = 'poimandres'
	let options: MagicMoveRenderOptions & MagicMoveDifferOptions = {
		duration: 1000,
		containerStyle: true,
	}
	let container: HTMLPreElement
	let highlighter: HighlighterCore
	let machine: ReturnType<typeof createMagicMoveMachine>
	let renderer: MagicMoveRenderer
	let ready = false
	let toggle = false

	async function init() {
		highlighter = await getHighlighter({ themes: ['poimandres'], langs: ['typescript'] })
		machine = createMagicMoveMachine(
			(code) => codeToKeyedTokens(highlighter, code, { lang, theme }),
			options
		)
		renderer = new MagicMoveRenderer(container)
		Object.assign(renderer.options, options)
		const result = machine.commit(code)
		renderer.render(result.current)
		ready = true
	}

	async function render() {
		const result = machine.commit(code)
		if (result.previous) renderer.replace(result.previous)
		console.log('start')
		await renderer.render(result.current)
		console.log('end')
	}

	function toggleAnimation() {
		toggle = !toggle
		toggle ? (code = 'let bool = true;') : (code = 'let bool;')
	}

	$: if (ready && code) {
		render()
	}

	onMount(init)
</script>

<svelte:window on:click={toggleAnimation} />

<pre bind:this={container} class="shiki-magic-move-container"></pre>
