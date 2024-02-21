<script lang="ts">
	import { Input } from '$lib/components/ui/input';
	import * as Table from '$lib/components/ui/table';
	// import { Label } from "$lib/components/ui/label";
	import { Button } from '$lib/components/ui/button';
	import csvtojson from 'csvtojson';
	import { json2csv } from 'json-2-csv';

	// to do
	let features: string[];
	let userData: any[] = [];
	let anonymizedStringArray: any[] = [];
	let anonymizedData: any[] = [];
	let jsonArray: object[] = [];
	let dataObject;

	let originalCSV: string | Blob;
	let anonymizedCSV;

	function readFile(file: Blob) {
		return new Promise((resolve, reject) => {
			const reader = new FileReader();
			reader.onload = () => resolve(reader.result);
			reader.onerror = reject;
			reader.readAsText(file);
		});
	}

	function objectToStringArray(objectArray: object[]) {
		return objectArray.map((obj) => {
			return Object.values(obj);
		});
	}

	function getFeatures(objectArray: object[]) {
		if (objectArray.length > 0) {
			return Object.keys(objectArray[0]);
		}
	}

	const handleGetFile = async (e: any) => {
		const file = e.target.files[0];
		originalCSV = file;

		try {
			const csvData = await readFile(file);
			jsonArray = await csvtojson().fromString(csvData as string);
			userData = objectToStringArray(jsonArray);
			features = getFeatures(jsonArray) as string[];
			// console.log(userData);
		} catch (err) {
			console.error(err);
		}
	};

	const handleDemoData = async () => {
		try {
			let response = await fetch('/demo/breast-cancer.csv');

			if (!response.ok) {
				throw new Error('Network response was not ok');
			}

			let csvData = await response.text();

			originalCSV = csvData;

			jsonArray = await csvtojson().fromString(csvData as string);
			userData = objectToStringArray(jsonArray);
			features = getFeatures(jsonArray) as string[];
		} catch (err) {
			console.error(err);
		}
	};

	async function convertStringToCSV(file: string) {
		const lines = file.split('\n');
		const headers = lines[0].split(',');
		const data = [];

		for (let i = 1; i < lines.length; i++) {
			const values = lines[i].split(',');
			const obj = JSON.parse(
				JSON.stringify(
					headers.reduce((prev, header, i) => {
						(prev as { [key: string]: any })[header] = values[i];
						return prev;
					}, {})
				)
			);
			data.push(obj);
		}

		return data;
	}

	async function convertObjectToCSV(dataObject: object[]) {
		return await json2csv(dataObject);
	}

	const handleAnonymizeData = (e: any) => {
		e.preventDefault();

		const formData = new FormData();
		formData.append('file', originalCSV);

		const url = 'https://anonymizer-mwnxl4smaa-el.a.run.app';

		fetch(url, {
			method: 'POST',
			body: formData
		})
			.then((response) => {
				if (!response.ok) {
					throw new Error('Network response was not ok');
				}

				return response.json();
			})
			.then(async (data) => {
				console.log(typeof data);
				console.log(data);

				anonymizedData = await convertStringToCSV(data);
				console.log(anonymizedData);

				anonymizedCSV = convertObjectToCSV(await anonymizedData);
				console.log(anonymizedCSV);

				anonymizedStringArray = objectToStringArray(anonymizedData);
				console.log(anonymizedStringArray);
			})
			.catch((error) => {
				console.error('There was a problem with your fetch operation:', error);
			});
	};

	// const handleFeatureSelect = (e) => {
	//   e.preventDefault();
	//   const formData =

	//   const url = 'https://trend-predictor-mwnxl4smaa-el.a.run.app';

	// 	fetch(url, {
	// 		method: 'POST',
	// 		body: formData
	// 	})
	// 		.then((response) => {
	// 			if (!response.ok) {
	// 				throw new Error('Network response was not ok');
	// 			}

	// 			return response.json();
	// 		})
	// }
</script>

<main class="flex flex-col items-center">
	<div class="mx-4 mt-8 flex min-h-fit w-11/12 flex-col items-center p-4">
		<div class="mb-4 flex w-full flex-col justify-between gap-1.5 sm:flex-row">
			<!-- <Label for="Dataset">Click here to upload your dataset</Label> -->
			<Input
				id="picture"
				type="file"
				placeholder="Upload csv files here"
				accept=".csv"
				on:input={handleGetFile}
				class="w-full flex-1"
			/>
			<Button on:click={handleAnonymizeData} class=" w-44">Anonymize the Data</Button>
			<Button on:click={handleDemoData} class=" w-44">Upload Demo Data</Button>
		</div>

		{#if userData.length}
			<h1 class="mb-4 mt-6 w-full text-center font-bold">Your Dataset</h1>
			<div class="max-h-96 w-full overflow-y-scroll object-cover p-4">
				<Table.Root class="overflow-x-visible">
					<Table.Caption>A list of your recent invoices.</Table.Caption>
					<Table.Header>
						<Table.Row>
							{#each features as feature, i (i)}
								<Table.Head class="font-bold">{feature}</Table.Head>
							{/each}
						</Table.Row>
					</Table.Header>
					<Table.Body>
						{#each userData as tuple, i (i)}
							<Table.Row>
								{#each features as feature, j (j)}
									<Table.Cell>{jsonArray[i][feature]}</Table.Cell>
								{/each}
							</Table.Row>
						{/each}
					</Table.Body>
				</Table.Root>
			</div>
		{/if}
		{#if anonymizedData.length > 0}
			<!-- to do -->
			<h1 class="mb-4 mt-6 w-full text-center font-bold">Anonymized Data</h1>
			<div class="max-h-96 w-full overflow-y-scroll border-2 border-slate-200 object-cover p-4">
				<Table.Root>
					<Table.Caption>A list of your recent invoices.</Table.Caption>
					<Table.Header>
						<Table.Row>
							{#each features as feature, i (i)}
								<Table.Head class="font-bold text-black">{feature}</Table.Head>
							{/each}
						</Table.Row>
					</Table.Header>
					<Table.Body>
						{#each anonymizedData as tuple, i (i)}
							<Table.Row>
								{#each features as feature, j (j)}
									<Table.Cell>{anonymizedData[i][feature]}</Table.Cell>
								{/each}
							</Table.Row>
						{/each}
					</Table.Body>
				</Table.Root>
			</div>
			<form class="flex w-full items-center space-x-2">
				<Input type="text" placeholder="Type the column name to use as the target" class="w-full" />
				<Button type="submit" class="w-48">Enter</Button>
			</form>
		{/if}
	</div>
</main>
