<script lang="ts">
	let willEncodeText = $state('');
	let willDecodeText = $state('');

	const bytesToBase64 = (bytes: Uint8Array): string => {
		return btoa(String.fromCodePoint(...bytes));
	}

	const base64ToBytes = (base64: string): Uint8Array => {
		return Uint8Array.from(atob(base64), (m) => m.codePointAt(0));
	}

	function handleEncodeButtonClick() {
		willDecodeText = bytesToBase64(new TextEncoder().encode(willEncodeText));
	}

	function handleDecodeButtonClick() {
		willEncodeText = new TextDecoder().decode(base64ToBytes(willDecodeText));
	}
</script>

<svelte:head>
	<title>base64</title>
	<meta name="description" content="base64 encode/decode" />
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.8.0/font/bootstrap-icons.css">
</svelte:head>

<article>
	<textarea id="will-encode-text" class="textarea textarea-bordered textarea-xs w-full max-w-xs" cols="40" rows="4" bind:value={willEncodeText}></textarea>
	<div class="base64-button-area">
		<buttom class="btn btn-xs btn-outline" onclick={handleEncodeButtonClick}><i class="bi bi-arrow-down-square-fill" />encode</buttom>
		<buttom class="btn btn-xs btn-outline" onclick={handleDecodeButtonClick}><i class="bi bi-arrow-up-square-fill" />decode</buttom>
	</div>
	<textarea id="will-decode-text" class="textarea textarea-bordered textarea-xs w-full max-w-xs" cols="40" rows="4" bind:value={willDecodeText}></textarea>
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
