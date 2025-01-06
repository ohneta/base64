<script lang="ts">
	let willEncodeText = $state('');
	let willDecodeText = $state('');

	const reader = new FileReader();	// 必須: "export const ssr = false;" at +page.tsファイル
	let dropTextareaFlag = 0;	// willEncodeText = 0 or willDecodeText = 1
	let dropFiles = $state([]);

	const bytesToBase64 = (bytes: Uint8Array): string => {
		return btoa(String.fromCodePoint(...bytes));
	}

	const base64ToBytes = (base64: string): Uint8Array => {
		return Uint8Array.from(atob(base64), (m) => m.codePointAt(0));
	}

	function handleEncodeButtonClick(): void {
		willDecodeText = bytesToBase64(new TextEncoder().encode(willEncodeText));
	}

	function handleDecodeButtonClick(): void {
		willEncodeText = new TextDecoder().decode(base64ToBytes(willDecodeText));
	}

	function handleClearEncodeButtonClick(): void {
		willEncodeText = '';
	}
	function handleClearDecodeButtonClick(): void {
		willDecodeText = '';
	}

	function handlerDragOver(evt: DragEvent) {
		evt.preventDefault();
	}

	// ondropイベント -- Encode textarea
	function handleFileDropEncode(evt: DragEvent) {
		dropTextareaFlag = 0;
		handleFileDrop(evt);
	}

	// ondropイベント -- Decode textarea
	function handleFileDropDecode(evt: DragEvent) {
		dropTextareaFlag = 1;
		handleFileDrop(evt);
	}

	// ondropイベント -- generic
	function handleFileDrop(evt: DragEvent) {
		evt.preventDefault();
		if (evt.dataTransfer.items) {
			[...evt.dataTransfer.items].forEach((item, i) => {
				if (item.kind === "file") {	// ドロップしたものがファイル("file")のみ有効
					const file = item.getAsFile();
					dropFiles[i] = file;
				}
			});
		} else {
			[...evt.dataTransfer.files].forEach((file, i) => {
				dropFiles[i] = file;
			});
		}
		reader.readAsText(dropFiles[0]);
	}

	reader.onload = (evt: ProgressEvent) => {
		if (dropTextareaFlag == 0) {
			willEncodeText = evt.target.result;
		} else
		if (dropTextareaFlag == 1) {
			willDecodeText = evt.target.result;
		}
	}

</script>

<svelte:head>
	<title>base64</title>
	<meta name="description" content="base64 encode/decode" />
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.8.0/font/bootstrap-icons.css">
</svelte:head>

<article>

	<buttom class="btn btn-xs btn-outline" onclick={handleClearEncodeButtonClick}><i class="bi bi-x-circle" />clear</buttom>
	<textarea 
		ondragover={handlerDragOver}
		ondrop={handleFileDropEncode}
		id="will-encode-text" class="textarea textarea-bordered textarea-xs w-full max-w-xs" cols="40" rows="4" bind:value={willEncodeText}
		placeholder="plane text or file"></textarea>

	<div class="base64-button-area">
		<buttom class="btn btn-xs btn-outline" onclick={handleEncodeButtonClick}><i class="bi bi-arrow-down-square-fill" />encode</buttom>
		<buttom class="btn btn-xs btn-outline" onclick={handleDecodeButtonClick}><i class="bi bi-arrow-up-square-fill" />decode</buttom>
	</div>

	<textarea
		ondragover={handlerDragOver}
		ondrop={handleFileDropDecode}
		id="will-decode-text" class="textarea textarea-bordered textarea-xs w-full max-w-xs" cols="40" rows="4" bind:value={willDecodeText}
		placeholder="base64 text or file"></textarea>
	<buttom class="btn btn-xs btn-outline" onclick={handleClearDecodeButtonClick}><i class="bi bi-x-circle" />clear</buttom>

</article>

<style>
article {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0;
}

.base64-button-area {
	flex-direction: column;
	justify-content: center;
	align-items: center;
	margin: 0.5em 0 0.5em 0;
}

</style>