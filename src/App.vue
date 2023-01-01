<script setup>
import { RouterView } from 'vue-router';
import HeaderView from './components/HeaderView.vue';
import FooterView from './components/FooterView.vue';
</script>

<template>

	<HeaderView />

	<div class="relative mx-auto max-w-7xl px-6 py-12">
		<RouterView />
	</div>

	<FooterView />

</template>
