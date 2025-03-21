<script setup>
import { onBeforeMount, ref } from "vue";
import { getItems } from "../libs/fetchUtils.js"
import { addItem } from '../libs/fetchUtils.js';
import { useRouter } from "vue-router";
import { useStatusStore } from '../stores/statusStore.js';
import { useNotiStore } from '../stores/notificationStore.js';
import { useLoginStore } from '../stores/loginStore.js';
import 'animate.css';

const statusStore = useStatusStore();
const statuses = statusStore.getStatuses();
const notiStore = useNotiStore();
const loginStore = useLoginStore();

const addStatus = ref({ id: "", name: "", description: "" })

const router = useRouter();

const getAllStatus = async () => {
    try {
        const data = await getItems(`${import.meta.env.VITE_BASE_URL}/v2/statuses`, loginStore.getToken());
        statuses.value = data;
    } catch (error) {
        console.error('Failed to fetch status:', error);
    }
};

const saveStatus = async () => {
    try {
        const isUnique = !statuses.value.some(status => status.name === addStatus.value.name);
        if (!isUnique) {
            notiStore.setNotificationMessage("Status name must be unique, please choose another name.");
            notiStore.setShowNotification(true);
            notiStore.setNotificationType("error");
            return;
        }
        if (addStatus.value.name === "") {
            notiStore.setNotificationMessage("Name cannot be empty");
            notiStore.setShowNotification(true);
            notiStore.setNotificationType("error");
            router.push({ name: 'StatusList' });
            return;
        }
        const newStatus = await addItem(`${import.meta.env.VITE_BASE_URL}/v2/statuses`, addStatus.value, loginStore.getToken());    
        if (!newStatus.status === 201) {
            throw new Error("Failed to add status");
        }
        statusStore.addStatus(newStatus);
        notiStore.setNotificationMessage(`The status "${addStatus.value.name}" has been added.`);
        notiStore.setShowNotification(true);
        notiStore.setNotificationType("success");
        router.push({ name: 'StatusList' });
        addStatus.value = { name: "", description: "" };
    } catch (error) {
        console.error('Error saving status:', error);
        notiStore.setNotificationMessage(`An error occurred, the status "${addStatus.value.name}" could not be added.`);
        notiStore.setShowNotification(true);
        notiStore.setNotificationType("error");
        router.push({ name: 'StatusList' });
    }
};

onBeforeMount(() => {
    getAllStatus();
});
</script>

<template>
    <!-- Add modal -->
    <div
        class="itbkk-modal-status bg-gray-900 bg-opacity-70 overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 flex justify-center items-center w-full inset-0 max-h-full">
        <div class="relative p-4 w-full max-w-xl max-h-full">
            <!-- Modal content -->
            <div class="relative bg-white rounded-lg shadow dark:bg-gray-700">
                <!-- Modal header -->
                <div class="flex items-center justify-between p-4 md:p-5 border-b rounded-t dark:border-gray-600">
                    <h3 class="text-lg font-semibold text-gray-900 dark:text-white">
                        Add New Status
                    </h3>
                    <button @click="router.push({ name: 'StatusList' })"
                        class="itbkk-button-cancel text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 ms-auto inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
                        data-modal-toggle="crud-modal">
                        <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
                            viewBox="0 0 14 14">
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6" />
                        </svg>
                    </button>
                </div>
                <!-- Modal body -->
                <form class="p-4 md:p-5">
                    <div class="grid gap-4 mb-4 grid-cols-2">
                        <div class="itbkk-status-name col-span-2">
                            <label for="name"
                                class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Name</label>
                            <input type="text" id="name" v-model.trim="addStatus.name" maxlength="50"
                                class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-600 dark:border-gray-500 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500"
                                placeholder="Type status name" required="">
                        </div>
                        <div class="itbkk-status-description col-span-2">
                            <label for="description"
                                class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">
                                Description</label>
                            <textarea id="description" rows="4" v-model.trim="addStatus.description" maxlength="200"
                                class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-600 dark:border-gray-500 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500 resize-none"
                                placeholder="Write status description here"></textarea>
                        </div>
                    </div>
                    <div @click="saveStatus" :class="{ 'cursor-not-allowed opacity-50': addStatus.name.trim() === '' }"
                        :disabled="addStatus.name.trim() === ''"
                        class="itbkk-button-confirm text-white inline-flex items-center bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
                        <svg class="me-1 -ms-1 w-5 h-5" fill="currentColor" viewBox="0 0 20 20"
                            xmlns="http://www.w3.org/2000/svg">
                            <path fill-rule="evenodd"
                                d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z"
                                clip-rule="evenodd"></path>
                        </svg>
                        Add new status
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>

<style scoped></style>