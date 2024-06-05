<script setup>
import InputLabel from './InputLabel.vue';
import InputError from './InputError.vue';
import PrimaryButton from './PrimaryButton.vue';
import TextInput from './TextInput.vue';
import { ref, watch } from 'vue'
import axios from 'axios'
import { getFingerprint, getFingerprintData } from '@thumbmarkjs/thumbmarkjs'

const BASE_API_URL = import.meta.env.VITE_BASE_API_URL;

const formSubmitted = ref(false)
const submitError = ref('')
const form = ref({
    tg_username: '',
    tg_first_name: '',
    tg_last_name: '',
    email: '',
    slotname: '',
    name_in_chat: '',
    whom_send: 'prize_for_me',
    fingerprint: '',
    fingerprint_data: '',
    ip_address: '',
    errors: {}
});
const tgWebapp = ref({})

const submit = () => {
    axios.post(BASE_API_URL + '/api/webapp/form', form.value, {headers: {
    }})
    .then(function (response) {
        console.log(response);
        formSubmitted.value = true;
    })
    .catch(function (error) {
        console.log(error);
        try {
            const data = JSON.parse(error.response); // Try to parse the response as JSON
            submitError.value = data.message || data
        } catch(err) {
            // The response wasn't a JSON object
            // Do your text handling here
            submitError.value = error
        }
    });
};

function getIpData () {
    axios.get(BASE_API_URL + '/api/webapp/user/ip')
    .then(function (response) {
        form.value.ip_address = response.data.ip
    }).catch(function (error) {
        console.log(error);
        submitError.value = error
    });
}

function initTelegramWepApp() {
    Telegram.WebApp.ready();

    const webApp = window.Telegram.WebApp;
    console.log(webApp)
    tgWebapp.value = webApp.initDataUnsafe

    form.value.tg_username = webApp.initDataUnsafe?.user?.username || ''
    form.value.tg_first_name = webApp.initDataUnsafe?.user?.first_name || ''
    form.value.tg_last_name = webApp.initDataUnsafe?.user?.last_name || ''
}

getFingerprint().then(function(fp) {
    console.log(fp);
    form.value.fingerprint = fp;
});

getFingerprintData().then((data) => {
    console.log(data);
    form.value.fingerprint_data = JSON.stringify(data);
})

initTelegramWepApp();
getIpData();

</script>

<template>
        <div v-if="formSubmitted">
            <p>Вы успешно добавлены в очередь!</p>
        </div>
        <div v-else-if="submitError">
            <InputError class="mt-2" :message="submitError" />
        </div>
        <form v-else @submit.prevent="submit">
            <div>
                <p>Ваш IP: {{ form.ip_address }}</p>
                <div>
                    <span>Ваши данные из Телеграмм:</span>
                    <p>{{ tgWebapp }}</p>
                </div>
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
                <InputLabel for="slotname" value="Рекомендуемый слот" />

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
                <InputLabel for="name_in_chat" value="Ваше имя в чате на платформе где вы сотрите стрим" />

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
                    <b class="block font-medium text-sm text-gray-700">Кому отправить выигрыш?</b>

                    <div class="mt-2">
                        <input type="radio" id="one" value="prize_for_me" v-model="form.whom_send">
                        <label for="one">Мне</label>
                    </div>
                    <div class="mt-2">
                        <input class="mt-2" type="radio" id="two" value="prize_for_romba" v-model="form.whom_send">
                        <label for="two">Ромба решает</label>
                    </div>
            </div>

            <div class="flex items-center justify-end mt-4">
                <PrimaryButton class="ms-4" :class="{ 'opacity-25': form.processing }" :disabled="form.processing">
                    Записаться в очередь
                </PrimaryButton>
            </div>
        </form>
</template>
