<script lang="ts">
	import {
		pipeline,
		env,
		ObjectDetectionPipeline,
		type ObjectDetectionPipelineOutput
	} from '@xenova/transformers';
	import { onMount } from 'svelte';
	env.allowLocalModels = false;

	let statusText = '';
	let detector: ObjectDetectionPipeline;

	// Reference the elements that we will need
	// const status = document.getElementById('status');
	// const fileUpload = document.getElementById('file-upload');
	let imageContainer: HTMLDivElement; // = document.getElementById('image-container');

	// Detect objects in the image
	const detect = async (img: HTMLImageElement) => {
		statusText = 'Analysing...';
		const output = (await detector(img.src, {
			threshold: 0.5,
			percentage: true
		})) as ObjectDetectionPipelineOutput;
		statusText = 'Ready';
		output.forEach(renderBox);
	};

	const handleFileUpload = (e: Event) => {
		// @ts-expect-error Property 'target' does not exist on type 'EventTarget'.
		if (!e?.target?.files) return;
		// @ts-expect-error Property 'target' does not exist on type 'EventTarget'.
		const file = e.target.files[0];
		if (!file) {
			return;
		}

		const reader = new FileReader();

		// Set up a callback when the file is loaded
		reader.onload = function (e2) {
			imageContainer.innerHTML = '';
			const image = document.createElement('img');
			// @ts-expect-error Property 'target' does not exist on type 'FileReader'.
			image.src = e2.target.result;
			image.style.width = '100%';
			imageContainer.appendChild(image);
			detect(image);
		};
		reader.readAsDataURL(file);
	};

	// Render a bounding box and label on the image
	const renderBox = ({
		box,
		label
	}: {
		box: { xmax: number; xmin: number; ymax: number; ymin: number };
		label: string;
	}) => {
		console.log(box, label);
		const { xmax, xmin, ymax, ymin } = box;

		// Generate a random color for the box
		const color =
			'#' +
			Math.floor(Math.random() * 0xffffff)
				.toString(16)
				.padStart(6, '0');

		// Draw the box
		const boxElement = document.createElement('div');
		boxElement.className = 'bounding-box';
		Object.assign(boxElement.style, {
			borderColor: color,
			left: 100 * xmin + '%',
			top: 100 * ymin + '%',
			width: 100 * (xmax - xmin) + '%',
			height: 100 * (ymax - ymin) + '%',
			position: 'absolute',
			boxSizing: 'border-box',
			borderWidth: '2px',
			borderStyle: 'solid'
		});

		// Draw label
		const labelElement = document.createElement('span');
		labelElement.textContent = label;
		labelElement.className = 'bounding-box-label';
		labelElement.style.backgroundColor = color;
		labelElement.style.color = 'white';
		labelElement.style.position = 'absolute';
		labelElement.style.fontSize = '12px';
		labelElement.style.marginTop = '-16px';
		labelElement.style.marginLeft = '-2px';
		labelElement.style.padding = '1px';

		boxElement.appendChild(labelElement);
		imageContainer.appendChild(boxElement);
	};

	onMount(async () => {
		// Create a new object detection pipeline
		// status.textContent = 'Loading model...';
		statusText = 'Loading model...';
		detector = await pipeline('object-detection', 'Xenova/detr-resnet-50');

		// status.textContent = 'Ready';
		statusText = 'Ready';
	});
</script>

<main class="container">
	{#if statusText === 'Ready'}
		<label class="custom-file-upload">
			<input id="file-upload" type="file" accept="image/*" on:change={handleFileUpload} />
			<img
				class="upload-icon"
				src="https://huggingface.co/datasets/Xenova/transformers.js-docs/resolve/main/upload-icon.png"
			/>
			Upload image
		</label>
	{/if}
	<p id="status">{statusText}</p>
	<div bind:this={imageContainer} id="image-container"></div>
</main>

<style>
	.container {
		margin: 40px auto;
		width: max(50vw, 400px);
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.custom-file-upload {
		display: flex;
		align-items: center;
		cursor: pointer;
		gap: 10px;
		border: 2px solid black;
		padding: 8px 16px;
		cursor: pointer;
		border-radius: 6px;
	}

	#file-upload {
		display: none;
	}

	.upload-icon {
		width: 30px;
	}

	#image-container {
		width: 100%;
		margin-top: 20px;
		position: relative;
	}


</style>
