<script setup>
import { ref } from 'vue'
import draggable from 'vuedraggable'
import AddonItem from './AddonItem.vue'

const stremioAPIBase = "https://api.strem.io/api/"

const stremioAuthKey = ref('')
let addons = ref([])

const dragging = false

function getCleansedStremioAuthKey() {
    return stremioAuthKey.value.replaceAll('"', '').trim();
}

function loadUserAddons() {
    const cleansedAuthKey = getCleansedStremioAuthKey()
    if (!cleansedAuthKey) {
        console.error('No auth key provided')
        return
    }

    alert('Loading addons')
    console.log('Loading addons...')

    const url = `${stremioAPIBase}addonCollectionGet`
    fetch(url, {
        method: 'POST',
        body: JSON.stringify({
            type: 'AddonCollectionGet',
            authKey: cleansedAuthKey,
            update: true,
        })
    }).then((resp) => {
        resp.json().then((data) => {
            console.log(data)
            addons.value = data.result.addons
        })
    }).catch((error) => {
        console.error('Error fetching user addons', error)
    })
}

function syncUserAddons() {
    const cleansedAuthKey = getCleansedStremioAuthKey()
    if (!cleansedAuthKey) {
        console.error('No auth key provided')
        return
    }
    console.log('Syncing addons...')

    const url = `${stremioAPIBase}addonCollectionSet`
    fetch(url, {
        method: 'POST',
        body: JSON.stringify({
            type: 'AddonCollectionSet',
            authKey: cleansedAuthKey,
            addons: addons.value,
        })
    }).then((resp) => {
        resp.json().then((data) => {
            if (!("result" in data) || data.result == null) {
                console.error("Sync failed: ", data)
                alert('Sync failed if unknown error')
                return
            } else if (!data.result.success) {
                alert("Failed to sync addons: " + data.result.error)
            } else {
                console.log("Sync complete: + ", data)
                alert('Sync complete!')
            }
        })
    }).catch((error) => {
        alert("Error syncing addons: " + error)
        console.error('Error fetching user addons', error)
    })
}

function removeAddon(idx) {
    addons.value.splice(idx, 1)
}


function getNestedObjectProperty(obj, path, defaultValue = null) {
    try {
        console.log('path', path, 'obj', obj, 'defaultValue', defaultValue)
        return path.split('.').reduce((acc, part) => acc && acc[part], obj)
    } catch (e) {
        return defaultValue
    }
}

</script>

<template>
    <section id="configure">
        <h2>Configure</h2>
        <form onsubmit="return false;">
            <fieldset>
                <legend>Step 0: Paste Stremio AuthKey</legend>
                <p class="grouped">
                    <input type="password" v-model="stremioAuthKey" placeholder="Paste Stremio AuthKey here...">
                    <button class="button primary" @click="loadUserAddons">
                        Load Addons
                    </button>
                </p>
            </fieldset>
            <fieldset id="form_step1">
                <legend>Step 1: Re-Order Addons</legend>
                <draggable :list="addons" item-key="transportUrl" class="sortable-list" ghost-class="ghost"
                    @start="dragging = true" @end="dragging = false">
                    <template #item="{ element, index }">
                        <AddonItem :name="element.manifest.name" :idx="index" :manifestURL="element.transportUrl"
                            :logoURL="element.manifest.logo"
                            :isDeletable="!getNestedObjectProperty(element, 'flags.protected', false)"
                            :isConfigurable="getNestedObjectProperty(element, 'manifest.behaviorHints.configurable', false)"
                            @delete-addon="removeAddon" />
                    </template>
                </draggable>
            </fieldset>
            <fieldset id="form_step2">
                <legend>Step 2: Sync Addons</legend>
                <button type="button" class="button primary icon" @click="syncUserAddons">Sync to Stremio
                    <img src="https://icongr.am/feather/loader.svg?size=16&amp;color=ffffff" alt="icon">
                </button>
            </fieldset>
        </form>

    </section>
</template>

<style scoped>
/* sortable list/addon list specifics */
.sortable-list {
    padding: 25px;
    background: #fff;
    border-radius: 7px;
    padding: 30px 25px 20px;
    box-shadow: 0 15px 30px rgba(0, 0, 0, 0.1);
}

.item.dragging {
    opacity: 0.6;
}

.item.dragging :where(.details, i) {
    opacity: 0;
}
</style>
