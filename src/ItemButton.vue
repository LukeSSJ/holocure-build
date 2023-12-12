<style scoped>
    .item {
        width: 80px;
        height: 80px;
        background: #507280;
        border: 4px solid white;
        cursor: pointer;
		position: relative;
    }
    .item:disabled {
        opacity: 0.4;
    }
    .item img {
        max-width: 100%;
        max-height: 100%;
    }
	.collab-image {
		position: absolute;
		right: -2px;
		bottom: -2px;
	}
</style>

<template>
	<button class="item" :title="itemName">
		<template v-if="item">
			<img v-if="item" :src="imageUrl(`/${props.type}/${item.icon}.webp`)" :alt="item.name">
			<img v-if="item && item.weapons" src="/Collab.webp" class="collab-image"/>
		</template>
		<slot v-else/>
	</button>
</template>

<script setup>
import {computed} from 'vue'
import {weapons, items} from './data.js'

const props = defineProps({
	item: Object,
	type: String,
})
const item = props.item

const itemName = computed(() => {
	if (!item) return ''

	let name = item.name
	if (item.weapons) {
		const requirements = item.weapons.map(id => weapons.find(w => w.id == id).name)
		if (requirements.length === 3) requirements.shift()
		name += "\n = " + requirements.join(" + ")
	}
	if (item.item) name += " + " + items.find(i => i.id === item.item).name
	return name
})

function imageUrl(src) {
	return new URL("/holocure-build" + src, import.meta.url)
}
</script>
