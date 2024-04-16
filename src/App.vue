<template>
	<div class="App">
		<header class="App-header">
			<img :src="logo" class="App-logo" alt="logo" />
			<p>Hello Vite + Vue!</p>
			<p>
				<button type="button" @click="incrementCount">
					count is: {{ count }}
				</button>
			</p>
			<p>Edit <code>App.vue</code> and save to test HMR updates.</p>
			<p>
				<a
					class="App-link"
					href="https://vuejs.org"
					target="_blank"
					rel="noopener noreferrer"
				>
					Learn Vue
				</a>
				|
				<a
					class="App-link"
					href="https://vitejs.dev/guide/features.html"
					target="_blank"
					rel="noopener noreferrer"
				>
					Vite Docs
				</a>
			</p>
		</header>
	</div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
// @ts-ignore
import logo from './logo.svg';
import './App.css';

const count = ref(0);

const incrementCount = () => {
	count.value++;
};
</script>

<style scoped>
/*  */
</style>
