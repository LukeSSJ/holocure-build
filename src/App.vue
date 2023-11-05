<style>
    body {
        margin: 0;
        padding: 2rem 1rem;
        background: #306080;
        color: white;
        font-size: 1.2rem;
    }
    .text-header {
        margin-bottom: 0.5rem;
    }
    .item-wrap {
        display: flex;
        flex-wrap: wrap;
        gap: 6px;
    }
    .item {
        width: 80px;
        height: 80px;
        background: #507280;
        border: 4px solid white;
        cursor: pointer;
    }
    .item:disabled {
        opacity: 0.4;
    }
    .item img {
        max-width: 100%;
    }
</style>

<template>
    <div>
        <button @click="saveBuild">Save</button>
        <button @click="resetBuild">Reset</button>
        <button @click="getUrl">Copy URL</button>
    </div>

    <br>

    <div>
        Character:
        <select v-model="active.character">
            <option value="">...</option>
            <option v-for="character in characters" :key="character.id" :value="character">{{ character.name }}</option>
        </select>
    </div>

    <br>

    <!--<div>Skills</div>-->

    <div>
        <div class="text-header">Weapons:</div>
        <div class="item-wrap">
            <button v-for="weapon in activeWeapons" :key="weapon.id" @click="removeWeapon(weapon)" class="item">
                <img :src="imageUrl(`/weapons/${weapon.icon}.webp`)" :alt="weapon.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        <div class="text-header">Items:</div>
        <div class="item-wrap">
            <button v-for="item in active.items" :key="item.id" @click="removeItem(item)" class="item">
                <img :src="imageUrl(`/items/${item.icon}.webp`)" :alt="item.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        <div class="text-header">Stamps:</div>
        <div class="item-wrap">
            <button v-for="stamp in active.stamps" :key="stamp.id" @click="removeStamp(stamp)" class="item">
                <img :src="imageUrl(`/stamps/${stamp.icon}.webp`)" :alt="stamp.name">
            </button>
        </div>
    </div>

    <br><br>

    <div>
        <div class="text-header">Available Weapons:</div>
        <div class="item-wrap">
            <button v-for="weapon in weapons" :key="weapon.id" @click="addWeapon(weapon)" :disabled="weaponDisabled(weapon)" class="item">
                <img :src="imageUrl(`/weapons/${weapon.icon}.webp`)" :alt="weapon.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        <div class="text-header">Available Items:</div>
        <div class="item-wrap">
            <button v-for="item in items" :key="item.id" @click="addItem(item)" :disabled="itemDisabled(item)" class="item">
                <img :src="imageUrl(`/items/${item.icon}.webp`)" :alt="item.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        <div class="text-header">Available Stamps:</div>
        <div class="item-wrap">
            <button v-for="stamp in stamps" :key="stamp.id" @click="addStamp(stamp)" :disabled="stampDisabled(stamp)" class="item">
                <img :src="imageUrl(`/stamps/${stamp.icon}.webp`)" :alt="stamp.name">
            </button>
        </div>
    </div>
</template>

<script setup>
    /*
        TODO:
        - Icons for characters
        - Styling
        - Show character skills
        - Super items
    */

    import {ref, reactive, computed} from 'vue'
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
            || (weapon.weapons && weapon.weapons.find(id => weaponsUsed[id]))
            || (weapon.item && active.items.find(i => i.id === weapon.item))
            || (weapon.item && active.weapons.find(w => w.item))
    }

    function itemDisabled(item) {
        return active.items.length >= 6
            || active.items.includes(item)
            || active.weapons.find(w => w.item === item.id)
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
