<script lang="ts">
	import Highlight, { type HighlightContext } from '@highlight-ai/app-runtime'
	import { saveAs } from 'file-saver'
	import QRCode from 'qrcode'
	import { onMount } from 'svelte'

	let text = 'https://google.com'
	let canvasEl: HTMLCanvasElement
	let suggestion = ''

	onMount(() => {
		createQRCode()

		if (typeof window.Highlight === 'undefined') {
			console.error('Highlight environment is not available')
			return
		}

		Highlight.app.addListener('onContext', (context: HighlightContext) => {
			console.log('Invoked')

			if (context.suggestion) {
				suggestion = context.suggestion
				text = suggestion

				createQRCode()
			}
		})
	})

	const createQRCode = async () => {
		await QRCode.toCanvas(canvasEl, text)
		canvasEl.style.width = '100%'
		canvasEl.style.height = 'auto'
	}

	const downloadPNG = async () => {
		canvasEl.toBlob((blob: any) => {
			saveAs(blob, 'qr-code.png')
		})
	}

	const downloadSVG = async () => {
		const svg = await QRCode.toString(text, {
			type: 'svg',
		})
		const blob = new Blob([svg])
		saveAs(blob, 'qr-code.svg')
	}
</script>

<article>
	<header>
		<h1>QR Code Generator</h1>
	</header>
	<label for="QR Code text">Enter your URL</label>
	<input type="text" bind:value={text} on:input={createQRCode} />
	<div class="grid">
		<canvas bind:this={canvasEl} style="width: 100%;" />
		<div class="flex">
			<button class="contrast" on:click={downloadPNG}>PNG</button>
			<button class="contrast" on:click={downloadSVG}>SVG</button>
		</div>
	</div>
</article>

<style>
	article {
		width: 100%;
		max-width: 480px;
		margin: 0 auto;
	}

	canvas {
		display: block;
		margin: 0 auto 1em;
		border-radius: 8px;
	}

	.flex {
		display: flex;
		flex-direction: column;
		gap: 0.8em;
		justify-content: center;
	}
</style>
