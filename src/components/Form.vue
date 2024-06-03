<script setup>
import InputLabel from './InputLabel.vue';
import InputError from './InputError.vue';
import PrimaryButton from './PrimaryButton.vue';
import TextInput from './TextInput.vue';
import { ref, watch } from 'vue'
import axios from 'axios'
import { getFingerprint, getFingerprintData } from '@thumbmarkjs/thumbmarkjs'

const BASE_API_URL = import.meta.env.VITE_BASE_API_URL;

const form = ref({
    tg_username: 'tg_username',
    tg_first_name: 'tg_first_name',
    tg_last_name: 'tg_last_name',
    email: '',
    slotname: '',
    name_in_chat: '',
    whom_send: 'prize_for_me',
    fingerprint: '',
    fingerprint_data: '',
    ip_addresses: [],
    errors: {}
});
const tgWebapp = ref({})

const submit = () => {
    axios.post(BASE_API_URL + '/form', form)
    .then(function (response) {
        console.log(response);
    })
    .catch(function (error) {
        console.log(error);
    });
};

async function getIpData () {
    axios.get(BASE_API_URL + '/webapp/user/ip')
    .then(function (response) {
        form.value.ip_addresses = response.data
    }).catch(function (error) {
        console.log(error);
    });
}

getFingerprint().then(function(fp) {
    console.log(fp);
    form.fingerprint = fp;
});

getFingerprintData().then((data) => {
    console.log(data);
    form.fingerprint_data = JSON.stringify(data);
})

function initTelegramWepApp() {
    Telegram.WebApp.ready();

    const webApp = window.Telegram.WebApp;
    console.log(webApp)
    tgWebapp.value = webApp.initData

    // form.value.tg_username = webApp.initDataUnsafe.user.username
    // form.value.tg_first_name = webApp.initDataUnsafe.user.first_name
    // form.value.tg_last_name = webApp.initDataUnsafe.user.last_name
}

initTelegramWepApp();
getIpData();

</script>

<template>
        <form @submit.prevent="submit">
            <div>
                <p>{{ form.ip_addresses }}</p>
                <div>{{ tgWebapp }}</div>
                <InputLabel for="email" value="Email" />

                <TextInput
                    id="email"
                    type="email"
                    class="mt-1 block w-full"
                    v-model="form.email"
                    required
                    autofocus
                    autocomplete="username"
                />

                <InputError class="mt-2" :message="form.errors.email" />
            </div>

            <div class="mt-4">
                <InputLabel for="slotname" value="Slotname" />

                <TextInput
                    id="slotname"
                    type="text"
                    class="mt-1 block w-full"
                    v-model="form.slotname"
                    required
                    autocomplete="current-slotname"
                />

                <InputError class="mt-2" :message="form.errors.slotname" />
            </div>

            <div class="mt-4">
                <InputLabel for="name_in_chat" value="Name in chat" />

                <TextInput
                    id="name_in_chat"
                    type="text"
                    class="mt-1 block w-full"
                    v-model="form.name_in_chat"
                    required
                    autocomplete="current-name_in_chat"
                />

                <InputError class="mt-2" :message="form.errors.name_in_chat" />
            </div>

            <div class="block mt-4">
                    <b class="block font-medium text-sm text-gray-700">Who send prize</b>

                    <input type="radio" id="one" value="prize_for_me" v-model="form.whom_send">
                    <label for="one">Me</label>
                    <br>
                    <input type="radio" id="two" value="prize_for_romba" v-model="form.whom_send">
                    <label for="two">Romba decides</label>
            </div>

            <div class="flex items-center justify-end mt-4">
                <PrimaryButton class="ms-4" :class="{ 'opacity-25': form.processing }" :disabled="form.processing">
                    Join the Queue
                </PrimaryButton>
            </div>
        </form>
</template>
