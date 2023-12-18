<style scoped>
	.container {
		position: relative;
	}
    .item {
        width: 80px;
        height: 80px;
		padding: 0.5rem;
        background: #507280;
        border: 4px solid white;
        cursor: pointer;
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
	.tooltip {
		display: none;
		position: absolute;
		left: 0;
		top: 82px;
		z-index: 1;
		pointer-events: none;
		width: max-content;
		padding: 0.5rem 1rem;
		background: rgba(0, 0, 0, 0.8);
		border: 2px solid white;
	}
	.container:hover .tooltip {
		display: block
	}
	.tooltip-heading {
		text-align: left;
	}
	.item-container {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		margin: 1rem 0;
	}
	.plus {
		font-size: 3rem;
	}
</style>

<template>
	<div class="container">
		<button class="item">
			<template v-if="item">
				<img v-if="item" :src="imageUrl(`/${props.type}/${item.icon}.webp`)" :alt="item.name">
				<img v-if="item && item.weapons" src="/Collab.webp" class="collab-image"/>
			</template>
			<slot v-else/>
		</button>
		<div class="tooltip">
			<div class="tooltip-heading">{{ item.name }}</div>

			<div v-if="item.weapons" class="item-container">
				<template v-for="weapon,i in requirements" :key="weapon">
					<div class="item">
						<img :src="weaponImage(weapon)"/>
					</div>
					<div v-if="i === 0" class="plus">+</div>
				</template>
				<template v-if="item.item">
					<div class="plus">+</div>
					<div class="item">
						<img :src="itemImage(item.item)"/>
					</div>
				</template>
			</div>
		</div>
	</div>
</template>

<script setup>
import {computed} from 'vue'
import {weapons, items} from './data.js'

const props = defineProps({
	item: Object,
	type: String,
})
const item = props.item

const requirements = computed(() => {
	if (!item) return ''

	const requirements = [...item.weapons]
	if (requirements.length === 3) requirements.shift()

	return requirements
})

function imageUrl(src) {
	return new URL("/holocure-build" + src, import.meta.url)
}

function weaponImage(id) {
	const weapon = weapons.find(w => w.id === id)
	return imageUrl(`/weapons/${weapon.icon}.webp`)
}

function itemImage(id) {
	const image = items.find(w => w.id === id)
	return imageUrl(`/items/${image.icon}.webp`)
}
</script>
