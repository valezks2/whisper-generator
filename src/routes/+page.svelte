<script lang="ts">
	import { onMount } from 'svelte';

	let text: string = '';
	let showButtons: boolean = false;
	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D;
	let currentYear: number = new Date().getFullYear();
	let img: HTMLImageElement;

	let fontSize: number = 90;
	let strokeColor: string = '#000000';
	let textX: number = 0;
	let textY: number = 0;

	function drawImageAndText() {
		if (!img?.src) return;

		canvas.width = img.naturalWidth;
		canvas.height = img.naturalHeight;
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

		const lines: string[] = text.trim().split('\n');
		if (lines[0]) {
			ctx.font = `${fontSize}px Upright, sans-serif`;
			ctx.fillStyle = 'white';
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';
			ctx.strokeStyle = strokeColor;
			ctx.lineWidth = fontSize / 7.5;
			ctx.lineJoin = 'round';

			const x = canvas.width / 2 + textX;
			const lineHeight = fontSize;
			const totalHeight = lines.length * lineHeight;
			let y = canvas.height / 2 - totalHeight / 2 + lineHeight / 2 + textY;

			lines.forEach((line) => {
				ctx.strokeText(line, x, y);
				ctx.fillText(line, x, y);
				y += lineHeight;
			});
		}
	}

	function handleFile(e: Event) {
		const input = e.target as HTMLInputElement;
		const file = input?.files?.[0];
		if (file && file.type.startsWith('image/')) {
			const reader = new FileReader();
			reader.onload = (ev: ProgressEvent<FileReader>) => {
				const result = ev.target?.result;
				if (typeof result === 'string') {
					img.src = result;
				}
			};
			reader.readAsDataURL(file);
		}
	}

	function changePicture() {
		showButtons = false;
		text = '';
		fontSize = 90;
		strokeColor = '#000000';
		textX = 0;
		textY = 0;
		if (img) img.src = '';

		if (ctx && canvas) {
			ctx.clearRect(0, 0, canvas.width, canvas.height);
			canvas.width = 0;
			canvas.height = 0;
		}
		showImageContainer(false);
	}

	function showImageContainer(shouldShow: boolean) {
		const container = document.querySelector('.image-container');
		if (container && container instanceof HTMLElement) {
			container.style.display = shouldShow ? 'block' : 'none';
		}
	}

	function download() {
		if (!img?.src) return;
		const dataURL = canvas.toDataURL('image/png');
		const link = document.createElement('a');
		link.href = dataURL;
		link.download = 'whisper-generator.png';
		link.click();
	}

	onMount(() => {
		img = new Image();
		ctx = canvas.getContext('2d') as CanvasRenderingContext2D;

		img.onload = () => {
			drawImageAndText();
			showButtons = true;
			showImageContainer(true);
		};
	});

	$: if (img?.src) {
		(text, fontSize, strokeColor, textX, textY);
		if (text.trim() === '' && textX === 0 && textY === 0) {
			ctx.clearRect(0, 0, canvas.width, canvas.height);
			ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
		} else {
			drawImageAndText();
		}
	}
</script>

<div class="flex flex-col min-h-screen font-upright bg-white">
	<header class="bg-[#202020] text-center">
		<a href="/" class="text-[3.8rem] text-white header-text m-0 no-underline select-none"
			>Whisper Generator</a
		>
	</header>

	<main class="flex-1 flex flex-col items-center justify-center px-4">
		<div
			class="w-full max-w-[500px] mx-auto bg-[#dedede] border-2 border-black p-6 mt-8 mb-8 flex flex-col gap-4"
		>
			<textarea
				class="h-[120px] w-full bg-[#FEFEFE] border-2 border-black font-upright text-[1.5rem] mt-4 p-4 resize-none transition-colors duration-300 placeholder:text-[#202020] focus:border-[#505050] focus:outline-none focus:placeholder-[#505050] focus:ring-0"
				bind:value={text}
				placeholder="Enter your text"
			></textarea>

			{#if showButtons}
				<div class="grid grid-cols-2 gap-4 p-4 bg-[#f0f0f0] border-2 border-black">
					<label class="flex flex-col gap-1">
						<span class="text-[1rem]">Size: {fontSize}px</span>
						<input
							type="range"
							min="20"
							max="250"
							bind:value={fontSize}
							class="accent-black cursor-pointer"
						/>
					</label>

					<label class="flex flex-col gap-1">
						<span class="text-[1rem]">Outline color:</span>
						<input
							type="color"
							bind:value={strokeColor}
							class="w-full mt-[4px] h-2 cursor-pointer appearance-none border-none bg-transparent [&::-webkit-color-swatch-wrapper]:p-0 [&::-webkit-color-swatch]:border-none [&::-webkit-color-swatch]:rounded-full [&::-moz-color-swatch]:border-none [&::-moz-color-swatch]:rounded-full"
						/>
					</label>

					<label class="flex flex-col gap-1">
						<span class="text-[1rem]">Vertical: {textY}</span>
						<input
							type="range"
							min={-(canvas?.height / 2 || 500)}
							max={canvas?.height / 2 || 500}
							bind:value={textY}
							class="accent-black cursor-pointer"
						/>
					</label>

					<label class="flex flex-col gap-1">
						<span class="text-[1rem]">Horizontal: {textX}</span>
						<input
							type="range"
							min={-(canvas?.width / 2 || 500)}
							max={canvas?.width / 2 || 500}
							bind:value={textX}
							class="accent-black cursor-pointer"
						/>
					</label>
				</div>
			{/if}

			{#if !showButtons}
				<div
					class="bg-white border-2 border-black text-center text-[1.5rem] text-[#202020] my-4 p-8 relative transition-colors duration-300 cursor-pointer drop-area hover:bg-[#f5f5f5] hover:border-[#505050] hover:text-[#505050]"
					role="button"
					tabindex="0"
					on:dragover|preventDefault
					on:drop|preventDefault={(e) => {
						const dataTransfer = e.dataTransfer;
						if (dataTransfer && dataTransfer.files) {
							const file = dataTransfer.files[0];
							if (file && file.type.startsWith('image/')) {
								const reader = new FileReader();
								reader.onload = (ev) => {
									const result = ev.target?.result;
									if (typeof result === 'string') img.src = result;
								};
								reader.readAsDataURL(file);
							}
						}
					}}
				>
					Drag or upload your image
					<input
						type="file"
						accept="image/*"
						class="absolute top-0 left-0 w-full h-full opacity-0 cursor-pointer"
						on:change={handleFile}
					/>
				</div>
			{/if}

			<div
				class="mt-3 mb-1 flex gap-4 justify-center flex-wrap"
				style="display:{showButtons ? 'flex' : 'none'}"
			>
				<button
					class="bg-white border-2 border-black font-upright text-[1.2rem] text-[#202020] px-4 py-2 transition-colors duration-300 hover:bg-[#f5f5f5] hover:border-[#505050] hover:text-[#505050] cursor-pointer"
					on:click={changePicture}
				>
					Change picture
				</button>
				<button
					class="bg-white border-2 border-black font-upright text-[1.2rem] text-[#202020] px-4 py-2 transition-colors duration-300 hover:bg-[#f5f5f5] hover:border-[#505050] hover:text-[#505050] cursor-pointer"
					on:click={download}
				>
					Download picture
				</button>
			</div>

			<div class="text-center mt-4 image-container" style="display:{img?.src ? 'block' : 'none'}">
				<canvas
					id="canvas"
					bind:this={canvas}
					class="max-w-full border-2 border-black mx-auto block"
				></canvas>
			</div>
		</div>
	</main>

	<footer class="bg-[#202020] text-white text-[1.35rem] text-center footer-text py-2 w-full">
		<div class="max-w-[700px] mx-auto">
			<div class="mb-1">
				<p class="m-0 text-white">
					Whisper Generator is inspired by the former
					<a
						href="https://en.wikipedia.org/wiki/Whisper_(app)"
						target="_blank"
						rel="noopener noreferrer"
						class="text-white underline decoration-gray-600 transition-colors duration-300 hover:text-gray-200"
						>Whisper app</a
					>
				</p>
				<span style="font-family: sans-serif;">&copy;</span>
				{currentYear} — Whisper Generator
			</div>
		</div>
	</footer>
</div>
