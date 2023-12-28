<style>
    * {
        box-sizing: border-box;
    }
    body {
        margin: 0;
        padding: 1rem;
		padding-bottom: 2rem;
        background: #306080;
        color: white;
        font-size: 1.2rem;
		font-family: 'Fira Sans';
    }
	a {
		color: inherit;
	}
    button, select {
        padding: 0.5rem;
        background: rgba(0, 0, 0, 0.5);
        color: white;
        border: 4px solid white;
        cursor: pointer;
        font-size: 1.2rem;
    }
    .flex {
        display: flex;
        gap: 2rem;
    }
    .button-wrap {
        display: flex;
        gap: 0.5rem;
    }
    .grid {
        display: grid;
        gap: 0.5rem;
    }
    .item-wrap {
        display: flex;
        flex-wrap: wrap;
        gap: 0.5rem;
    }
    .text-header {
        margin-bottom: 0.5rem;
    }
	.footer {
		margin-top: 4rem;
		font-size: 1rem;
		color: #d5d5d5;
	}
</style>
<template>
    <div class="button-wrap">
        <button @click="resetBuild">Reset</button>
        <button @click="getUrl">Copy URL</button>
    </div>
    <br>

    <div class="flex">
        <div>
            <div>
                <select v-model="active.character">
                    <option value="">...</option>
                    <option v-for="character in characters" :key="character.id" :value="character">{{ character.name }}</option>
                </select>
            </div>
            <br>
            <div class="grid">
                <div class="item-wrap">
					<ItemButton
						v-for="weapon in activeWeapons"
						:key="weapon.name"
						:item="weapon"
						@click="removeWeapon(weapon)"
						type="weapons"
					/>
                    <ItemButton v-for="i in (6 - activeWeapons.length)" :key="i">
                        <img src="/Weapon.webp" alt="">
                    </ItemButton>
                </div>
                <div class="item-wrap">
					<ItemButton
						v-for="item in active.items"
						:key="item.id"
						:item="item"
						@click="removeItem(item)"
						type="items"
					/>
                    <ItemButton v-for="i in (6 - active.items.length)" :key="i">
                        <img src="/Item.webp" alt="">
					</ItemButton>
                </div>
                <div class="item-wrap">
                    <ItemButton
						v-for="stamp in active.stamps"
						:key="stamp.id"
						:item="stamp"
						@click="removeStamp(stamp)"
						type="stamps"
					/>
                    <ItemButton v-for="i in (3 - active.stamps.length)" :key="i" />
                </div>
            </div>
        </div>
        <div>
            <img v-if="active.character" :src="imageUrl(`/portraits/${active.character.id}.webp`)" />
        </div>
    </div>

    <br><br>

    <div>
        <div class="text-header">Available Weapons:</div>
        <div class="item-wrap">
            <ItemButton
				v-for="weapon in weapons"
				:key="weapon.id"
				@click="addWeapon(weapon)"
				:item="weapon"
				:disabled="weaponDisabled(weapon)"
				type="weapons"
			/>
        </div>
    </div>

    <br>

    <div>
        <div class="text-header">Available Items:</div>
        <div class="item-wrap">
			<ItemButton
				v-for="item in items"
				:key="item.id"
				:item="item"
				@click="addItem(item)"
				:disabled="itemDisabled(item)"
				type="items"
			/>
        </div>
    </div>

    <br>

    <div>
        <div class="text-header">Available Stamps:</div>
        <div class="item-wrap">
			<ItemButton
				v-for="stamp in stamps"
				:key="stamp.id"
				@click="addStamp(stamp)"
				:disabled="stampDisabled(stamp)"
				:item="stamp"
				type="stamps"
			/>
        </div>
    </div>

	<div class="footer">
		Check out the source code <a href="https://github.com/LukeSSJ/holocure-build" target="_blank">here</a>
	</div>
</template>
<script setup>
	import ItemButton from './ItemButton.vue'

    import {ref, reactive, computed, watch} from 'vue'
    import {characters, weapons, items, stamps} from './data.js'

    const active = reactive({
        character: '',
        weapons: [],
        items: [],
        stamps: [],
    })

    const activeWeapons = computed(() => {
        return [active.character.weapon].concat(active.weapons).filter(w => w)
    })

    const weaponsUsed = {}
    let collabs = 0

    loadBuild()

	watch(active, saveBuild);

    function addWeapon(weapon) {
        active.weapons.push(weapon)
        weaponsUsed[weapon.id] = true
        if (weapon.weapons) {
            weapon.weapons.map(id => weaponsUsed[id] = true)
            collabs += 1
        }
    }

    function addItem(item) {
        active.items.push(item)
    }

    function addStamp(stamp) {
        active.stamps.push(stamp)
    }

    function removeWeapon(weapon) {
        arrayRemove(active.weapons, weapon)
        weaponsUsed[weapon.id] = false
        if (weapon.weapons) {
            weapon.weapons.map(id => weaponsUsed[id] = false)
            collabs -= 1
        }
    }

    function removeItem(item) {
        arrayRemove(active.items, item)
    }

    function removeStamp(stamp) {
        arrayRemove(active.stamps, stamp)
    }

    function weaponDisabled(weapon) {
        return active.weapons.length >= 5
            || active.weapons.includes(weapon)
            || weaponsUsed[weapon.id]
            || (weapon.weapons && collabs >= 4)
            || (weapon.weapons && !!weapon.weapons.find(id => weaponsUsed[id]))
            || (weapon.item && !!active.items.find(i => i.id === weapon.item))
            || (weapon.item && !!active.weapons.find(w => w.item))
    }

    function itemDisabled(item) {
        return active.items.length >= 6
            || active.items.includes(item)
            || !!active.weapons.find(w => w.item === item.id)
    }

    function stampDisabled(stamp) {
        return active.stamps.length >= 3
            || active.stamps.includes(stamp)
    }

    function arrayRemove(array, item) {
        const index = array.indexOf(item)
        if (index !== -1) array.splice(index, 1)
    }

    function imageUrl(src) {
        return new URL("/holocure-build" + src, import.meta.url)
    }

    function saveBuild() {
        const build = {
            character: active.character?.id ?? "",
            weapons: active.weapons.map(w => w.id),
            items: active.items.map(i => i.id),
            stamps: active.stamps.map(s => s.id),
        }
        localStorage.setItem("build", JSON.stringify(build))
    }

    function loadBuild() {
        const params = new URLSearchParams(location.search)
        if (params.size) {
            active.character = characters.find(c => c.id === params.get("c")) || ""
            active.weapons = weapons.filter(w => params.get("w")?.split(",").includes(w.id))
            active.items = items.filter(i => params.get("i")?.split(",").includes(i.id))
            active.stamps = stamps.filter(s => params.get("s")?.split(",").includes(s.id))
            return
        }

        let build = localStorage.getItem("build")
        if (build) {
            try {
                build = JSON.parse(build)
                active.character = characters.find(c => c.id === build.character) || ""
                active.weapons = weapons.filter(w => build.weapons.includes(w.id))
                active.items = items.filter(i => build.items.includes(i.id))
                active.stamps = stamps.filter(s => build.stamps.includes(s.id))
            } catch(e) {}
        }
    }

    function resetBuild() {
        if (confirm("Reset Build?")) {
            active.character = ""
            active.weapons = []
            active.items = []
            active.stamps = []
        }
    }

    function getUrl() {
        let character = active.character?.id ?? ""
        let weapons = active.weapons.map(w => w.id).join(",")
        let items = active.items.map(i => i.id).join(",")
        let stamps = active.stamps.map(s => s.id).join(",")
        let url = `${location.origin}${location.pathname}?c=${character}&w=${weapons}&i=${items}&s=${stamps}`

        navigator.clipboard.writeText(url)
            .then(() => alert("Copied to clipboard"))
    }
</script>
