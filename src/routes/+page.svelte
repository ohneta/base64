<script lang="ts">
	import { onMount } from 'svelte';

	let targetEncode = 'text';	// 対象のデータ形式 text, image

	let willEncodeText = $state('');
	let willDecodeText = $state('');
	let willEncodeImage = $state('');

	let checkedAddDataURL = $state(false);

	const reader = new FileReader();	// 必須: "export const ssr = false;" at +page.tsファイル
	let dropTextareaFlag = 0;	// willEncodeText = 0 or willDecodeText = 1
	let dropFiles = $state([]);

	let blobDataBase64 = $state('');
	let dataMIME = $state('');
	let dataURL = $state('');

	let message = $state('');

	//--------
	const bytesToBase64 = (bytes: Uint8Array): string => {
		return btoa(String.fromCodePoint(...bytes));
	}

	const base64ToBytes = (base64: string): Uint8Array => {
		return Uint8Array.from(atob(base64), (m) => m.codePointAt(0));
	}

	//--------
	function handleEncodeButtonClick(): void {
		message = '';
		let tmpDecodeText = '';

		if (targetEncode == 'text') {
			tmpDecodeText = bytesToBase64(new TextEncoder().encode(willEncodeText));
			dataURL = 'data:text/plain;base64';
		} else
		if (targetEncode == 'image') {
			[blobDataBase64, dataMIME, dataURL] = blog2Data(willEncodeImage);
			tmpDecodeText = blobDataBase64;
		}

		willDecodeText = '';
		if (checkedAddDataURL) {
			willDecodeText = dataURL + ',';
		}
		willDecodeText += tmpDecodeText;
	}

	function handleDecodeButtonClick(): void {
		message = '';
		if (!checkedAddDataURL) {
			// dataURLを使わない場合は強制的にwillEncodeTextにデコードデータを表示
			try {
				willEncodeText = new TextDecoder().decode(base64ToBytes(willDecodeText));
			} catch (error) {
				message = "incorrect decode string";
				willEncodeText = '';
			}
			targetEncode = 'text';
			displayData(true, false);

		} else {
			// dataURLを使う場合はMIME毎に処理
			let [decodeBlobDataBase64, decodeDataMIME, decodeDataURL] = blog2Data(willDecodeText);
			if (decodeDataMIME == 'text/plain') {
				try {
					willEncodeText = new TextDecoder().decode(base64ToBytes(decodeBlobDataBase64));
				} catch (error) {
					message = "incorrect decode string";
					willEncodeText = '';
				}
				targetEncode = 'text';
				displayData(true, false);
			} else
			if (	 (decodeDataMIME == 'image/jpeg')
					|| (decodeDataMIME == 'image/png')	) {
				willEncodeImage = decodeDataURL + ',' + decodeBlobDataBase64;
				targetEncode = 'image';
				displayData(false, true);
			}
		}
	}

	function handleClearEncodeButtonClick(): void {
		message = '';
		willEncodeText = '';

		// クリアしたらエンコード形式は強制的にtext
		targetEncode = 'text';
		displayData(true, false);
	}
	function handleClearDecodeButtonClick(): void {
		message = '';
		willDecodeText = '';
	}

	//--------
	function handlerDragOver(evt: DragEvent) {
		evt.preventDefault();
	}

	// dragenter
	function handlerDragEnter(evt: DragEvent) {
		evt.target.style.border = "solid";
	}
	// dragleave
	function handlerDragLeave(evt: DragEvent) {
		evt.target.style.border = "";
	}

	// drop -- Encode textarea
	function handleFileDropEncode(evt: DragEvent) {
		dropTextareaFlag = 0;
		handleFileDrop(evt);
		evt.target.style.border = "";
	}

	// drop -- Decode textarea
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

//console.log(dropFiles[0].size);
		// if (dropFiles[0].type == 'text/plain') {
		// 	reader.readAsText(dropFiles[0]);	// テキストファルの読み込み
		// } else {
		// 	reader.readAsDataURL(dropFiles[0]);
		// }
		reader.readAsDataURL(dropFiles[0]);
	}

	/**
	 * FileReaderのresultを分解する
	 * @param fsResult string
	 * @return array [blobData, MIME, dataURL]
	 */
	function blog2Data(fsResult: string) {
		let blobDatas = '';
		let dataURL = '';
		let dataMIME = '';
		try {
			blobDatas = fsResult.split(',');
			dataURL = blobDatas[0];
			dataMIME = blobDatas[0].split(':')[1].split(';')[0];
		} catch (error) {
			message = "must set dataURL";
			blobDatas = '';
			dataURL = '';
			dataMIME = '';
		}

		return [blobDatas[1], dataMIME, dataURL];
	}

	reader.onload = (evt: ProgressEvent) => {
		[blobDataBase64, dataMIME, dataURL] = blog2Data(reader.result);

		if (dropTextareaFlag == 0)			willEncodeText = '';
		else if (dropTextareaFlag == 1)	willDecodeText = '';

		if (dataMIME == 'text/plain') {
			targetEncode = 'text';
			displayData(true, false);
			if (dropTextareaFlag == 0) {
				willEncodeText = new TextDecoder().decode(base64ToBytes(blobDataBase64));
			} else
			if (dropTextareaFlag == 1) {
				willDecodeText = evt.target.result;
			}
		} else
		if (	 (dataMIME == 'image/jpeg')
				|| (dataMIME == 'image/png')	) {
			if (dropTextareaFlag == 0) {
				willEncodeImage = reader.result;	// FileReaderの生データを設定する
				targetEncode = 'image';
				displayData(false, true);
			// } else
			// if (dropTextareaFlag == 1) {
			// 	willDecodeText = evt.target.result;
			}
		}

	}

	/**
	 * エンコードするデータの種類によって表示する要素をオンオフする
	 * @param textFlag: boolean  テキストを表示する
	 * @param imageFlag: boolean 画像を表示する
	 */
	function displayData(textFlag: boolean, imageFlag: boolean): void {
		document.getElementById('will-encode-text').style.display = textFlag ? '' : 'none';
		document.getElementById('will-encode-image').style.display = imageFlag ? '' : 'none';
	}

	onMount(() => {
		displayData(true, false);
	});

</script>

<svelte:head>
	<title>base64</title>
	<meta name="description" content="base64 encode/decode" />
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.8.0/font/bootstrap-icons.css">
</svelte:head>

<article>

	<buttom class="btn btn-xs btn-outline" onclick={handleClearEncodeButtonClick}><i class="bi bi-x-circle" />clear</buttom>
	<textarea 
		ondragenter={handlerDragEnter}
		ondragleave={handlerDragLeave}
		ondragover={handlerDragOver}
		ondrop={handleFileDropEncode}
		id="will-encode-text" class="textarea textarea-bordered textarea-xs w-full max-w-xs" cols="40" rows="4" bind:value={willEncodeText}
		placeholder="plane text or file (text/image)"></textarea>
	<img
		src="{willEncodeImage}"
		id="will-encode-image" 
		ondragenter={handlerDragEnter}
		ondragleave={handlerDragLeave}
		ondragover={handlerDragOver}
		ondrop={handleFileDropEncode}
		alt="will encode image"
	/>

	<div class="base64-button-area">

		<div class="form-check">
			<input class="form-check-input" type="checkbox" value="" id="check-add-dataURL" bind:checked={checkedAddDataURL}>
			<label class="form-check-label" for="check-add-dataURL">
				use dataURL
			</label>
		</div>

		<buttom class="btn btn-xs btn-outline" onclick={handleEncodeButtonClick}><i class="bi bi-arrow-down-square-fill" />encode</buttom>
		<buttom class="btn btn-xs btn-outline" onclick={handleDecodeButtonClick}><i class="bi bi-arrow-up-square-fill" />decode</buttom>
	</div>

	<textarea
		ondragenter={handlerDragEnter}
		ondragleave={handlerDragLeave}
		ondragover={handlerDragOver}
		ondrop={handleFileDropDecode}
		id="will-decode-text" class="textarea textarea-bordered textarea-xs w-full max-w-xs" cols="40" rows="4" bind:value={willDecodeText}
		placeholder="base64 text or file"></textarea>
	<buttom class="btn btn-xs btn-outline" onclick={handleClearDecodeButtonClick}><i class="bi bi-x-circle" />clear</buttom>

{#if 	message !== ''}
<div class="error-message-area">
	Error: {message}
</div>
{/if}
</article>


<!--
<div>
	<h4>check-image</h4>
	<img src={willDecodeText} />
</div>
-->

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

#will-encode-image {
	width: 320px;
}

.error-message-area {
	display: flex;
	justify-content: center;
	align-items: center;

	background-color: #0cd5d1;
	color: #a8658b;
	width: 320px;
	margin-top: 0.5em;
	padding: 0.5em 1em 0.5em 1em;
}


</style>