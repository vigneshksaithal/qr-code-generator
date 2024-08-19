<script lang="ts">
import Highlight, { type HighlightContext } from '@highlight-ai/app-runtime'
import { saveAs } from 'file-saver'
import QRCode from 'qrcode'
import { onMount } from 'svelte'

let text = 'https://highlight.ing'
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

	/* Resize the canvas to take full available width */
	canvasEl.style.width = '100%'
	canvasEl.style.height = 'auto'
}

const downloadPNG = () => {
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
		<p>
			Invoke the app via Highlight chat, paste the link in the chat for instant
			QR code.
		</p>
	</header>
	<div class="grid">
		<canvas bind:this={canvasEl} />
	</div>
	<input type="text" bind:value={text} on:input={createQRCode} />
	<footer class="grid">
		<button class="contrast" on:click={downloadPNG}>PNG</button>
		<button class="contrast" on:click={downloadSVG}>SVG</button>
	</footer>
</article>

<style>
article {
	width: 100%;
	max-width: 400px;
	margin: 0 auto;
}

canvas {
	display: block;
	margin: 0 auto 1em;
	max-width: 200px;
	padding: 0em;
	border: 2px solid #e2e8f0;
	border-radius: 6px;
}

input {
	margin: 0;
}
</style>
