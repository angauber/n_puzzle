<template>
	<div id="app">
		<puzzle/>
	</div>
</template>

<script>
import puzzle from './components/puzzle.vue'

	export default {
		name: 'app',
		components: {
			puzzle
		}
	}
</script>

<style>
#app {
	font-family: 'Avenir', Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
