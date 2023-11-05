<style>
    body {
        background: #306080;
        color: white;
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
        Weapons:
        <span v-if="active.character">{{ active.character.weapon }}</span>
        <div class="item-wrap">
            <button v-for="weapon in active.weapons" :key="weapon.id" @click="removeWeapon(weapon)" class="item">
                <img :src="imageUrl(`/weapons/${weapon.icon}.webp`)" :alt="weapon.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        Items:
        <div class="item-wrap">
            <button v-for="item in active.items" :key="item.id" @click="removeItem(item)" class="item">
                <img :src="imageUrl(`/items/${item.icon}.webp`)" :alt="item.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        Stamps:
        <div class="item-wrap">
            <button v-for="stamp in active.stamps" :key="stamp.id" @click="removeStamp(stamp)" class="item">
                <img :src="imageUrl(`/stamps/${stamp.icon}.webp`)" :alt="stamp.name">
            </button>
        </div>
    </div>

    <br><br>

    <div>
        Available Weapons:
        <div class="item-wrap">
            <button v-for="weapon in weapons" :key="weapon.id" @click="addWeapon(weapon)" :disabled="weaponDisabled(weapon)" class="item">
                <img :src="imageUrl(`/weapons/${weapon.icon}.webp`)" :alt="weapon.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        Available Items:
        <div class="item-wrap">
            <button v-for="item in items" :key="item.id" @click="addItem(item)" :disabled="itemDisabled(item)" class="item">
                <img :src="imageUrl(`/items/${item.icon}.webp`)" :alt="item.name">
            </button>
        </div>
    </div>

    <br>

    <div>
        Available Stamps:
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
        - Load build from URL
        - Icons for characters
        - Super items
        - Show character skills
        - Styling
    */

    import {ref, reactive} from 'vue'
    import {characters, weapons, items, stamps} from './data.js'

    const active = reactive({
        character: '',
        weapons: [],
        items: [],
        stamps: [],
    })

    const weaponsUsed = {}

    loadBuild()

    function addWeapon(weapon) {
        if (active.weapons.length < 5 && !active.weapons.includes(weapon)) {
            active.weapons.push(weapon)
            weaponsUsed[weapon.id] = true
            if (weapon.weapons) weapon.weapons.map(id => weaponsUsed[id] = true)
        }
    }

    function addItem(item) {
        if (active.items.length < 6 && !active.items.includes(item)) active.items.push(item)
    }

    function addStamp(stamp) {
        if (active.stamps.length < 3 && !active.stamps.includes(stamp)) active.stamps.push(stamp)
    }

    function removeWeapon(weapon) {
        arrayRemove(active.weapons, weapon)
        weaponsUsed[weapon.id] = false
        if (weapon.weapons) weapon.weapons.map(id => weaponsUsed[id] = false)
    }

    function removeItem(item) {
        arrayRemove(active.items, item)
    }

    function removeStamp(stamp) {
        arrayRemove(active.stamps, stamp)
    }

    function weaponDisabled(weapon) {
        return active.weapons.includes(weapon)
            || weaponsUsed[weapon.id]
            || (weapon.weapons && weapon.weapons.find(id => weaponsUsed[id]))
            || (weapon.item && active.items.find(i => i.id === weapon.item))
            || (weapon.item && active.weapons.find(w => w.item))
    }

    function itemDisabled(item) {
        return active.items.includes(item) || active.weapons.find(w => w.item === item.id)
    }

    function stampDisabled(stamp) {
        return active.stamps.includes(stamp)
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
