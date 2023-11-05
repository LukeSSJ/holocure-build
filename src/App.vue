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
        <button v-for="weapon in active.weapons" :key="weapon.id" @click="removeWeapon(weapon)">{{ weapon.name }}</button>
    </div>

    <br>

    <div>
        Items:
        <button v-for="item in active.items" :key="item.id" @click="removeItem(item)">{{ item.name }}</button>
    </div>

    <br>

    <div>
        Stamps:
        <button v-for="stamp in active.stamps" :key="stamp.id" @click="removeStamp(stamp)">{{ stamp.name }}</button>
    </div>

    <br><br>

    <div>
        Available Weapons:
        <button v-for="weapon in weapons" :key="weapon.id" @click="addWeapon(weapon)" :disabled="weaponDisabled(weapon)">{{ weapon.name }}</button>
    </div>

    <br>

    <div>
        Available Items:
        <button v-for="item in items" :key="item.id" @click="addItem(item)" :disabled="itemDisabled(item)">{{ item.name }}</button>
    </div>

    <br>

    <div>
        Available Stamps:
        <button v-for="stamp in stamps" :key="stamp.id" @click="addStamp(stamp)" :disabled="stampDisabled(stamp)">{{ stamp.name }}</button>
    </div>
</template>

<script setup>
    /*
        TODO:
        - Load build from URL
        - Super items
        - Show character skills
        - Icons for characters, weapons and items
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

    loadBuild()

    function addWeapon(weapon) {
        if (active.weapons.length < 5 && !active.weapons.includes(weapon)) active.weapons.push(weapon)
    }

    function addItem(item) {
        if (active.items.length < 6 && !active.items.includes(item)) active.items.push(item)
    }

    function addStamp(stamp) {
        if (active.stamps.length < 3 && !active.stamps.includes(stamp)) active.stamps.push(stamp)
    }

    function removeWeapon(weapon) {
        arrayRemove(active.weapons, weapon)
    }

    function removeItem(item) {
        arrayRemove(active.items, item)
    }

    function removeStamp(stamp) {
        arrayRemove(active.stamps, stamp)
    }

    function weaponDisabled(weapon) {
        return active.weapons.includes(weapon)
            || weapon.weapons?.find(id => active.weapons.find(w => w.id === id))
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
        console.log(url)
    }
</script>
