<template>
	<div>
		<input type="text" v-model="inputText" placeholder="Enter text" />
		<button @click="postData">Submit</button>
	</div>
</template>

<script lang="ts">
import axios from "axios";

export default class MyComponent {
	inputText: string = "";

	postData() {
		axios
			.post("https://example.com/api/endpoint", {
				data: this.inputText,
			})
			.then((response) => {
				console.log("Data posted successfully!", response.data);
				// Perform any additional actions on successful response
			})
			.catch((error) => {
				console.error("Error posting data:", error);
				// Handle error scenario
			});
	}
}
</script>
