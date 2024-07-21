<script setup lang="ts">
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
import {
  getFirestore,
  collection,
  getDocs,
  setDoc,
  doc,
  getDoc,
  DocumentSnapshot,
  type DocumentData,
} from "firebase/firestore";
import { onBeforeMount, reactive, ref, type Ref } from "vue";
import {
  TransitionRoot,
  TransitionChild,
  Dialog,
  DialogPanel,
  DialogTitle,
  Listbox,
  ListboxButton,
  ListboxOptions,
  ListboxOption,
} from "@headlessui/vue";
import { CheckIcon, ChevronUpDownIcon } from "@heroicons/vue/20/solid";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyCKyeCViiF3uS0EIzZ1Jd730NkP3O8mmJM",
  authDomain: "crystal-ranker.firebaseapp.com",
  projectId: "crystal-ranker",
  storageBucket: "crystal-ranker.appspot.com",
  messagingSenderId: "679066407394",
  appId: "1:679066407394:web:a80be1832791830cd8a9bb",
  measurementId: "G-K0R8K6SH6T",
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
const db = getFirestore(app);

let players: any = ref([]);

const isOpen = ref(false);

function closeModal() {
  isOpen.value = false;
}
function openModal() {
  isOpen.value = true;
}

const playstyles = [
  { id: 1, name: "Anything" },
  { id: 2, name: "Hole" },
  { id: 3, name: "Hole(Anchor Spammer)" },
  { id: 4, name: "Hole(Digger)" },
  { id: 5, name: "Surface" },
  { id: 6, name: "Surface(Pearl Abuser)" },
  { id: 7, name: "Runner" },
];

let selectedPlaystyle = ref(playstyles[0]);

onBeforeMount(async () => {
  players.value = await (
    await getDocs(collection(db, "Demo"))
  ).docs.map((e) => e.data());
});

interface addDataInterface {
  name: string;
  details: string;
  overall: number | null;
  surface: number | null;
  hole: number | null;
}

let addData: addDataInterface = reactive({
  name: "",
  details: "",
  overall: null,
  surface: null,
  hole: null,
});

const submitUser = async () => {
  addData.name = addData.name.trim();
  addData.details = addData.details.trim();
  let error = false;
  if (!addData.name) {
    error = true;
    console.log("Missing Name");
  }
  if (addData.overall == null || typeof addData.overall !== "number") {
    error = true;
    console.log("Missing Overall");
  }
  if (!addData.surface == null || typeof addData.surface !== "number") {
    error = true;
    console.log("Missing Surface");
  }
  if (!addData.hole == null || typeof addData.hole !== "number") {
    error = true;
    console.log("Missing Hole");
  }
  if (!selectedPlaystyle.value.name) {
    error = true;
    console.log("Missing Playstyle?");
  }
  if (error) return;

  if (addData.overall === null || addData.overall < 0 || addData.overall > 10) {
    error = true;
    console.log("Overall must be within 0 and 10");
  }
  if (addData.surface === null || addData.surface < 0 || addData.surface > 10) {
    error = true;
    console.log("Overall must be within 0 and 10");
  }
  if (addData.hole === null || addData.hole < 0 || addData.hole > 10) {
    error = true;
    console.log("Overall must be within 0 and 10");
  }

  if (error) return;

  await setDoc(doc(db, "Demo", addData.name.toLowerCase()), {
    username: addData.name,
    details: addData.details,
    overall: addData.overall,
    surface: addData.surface,
    hole: addData.hole,
    playstyle: selectedPlaystyle.value.name,
  });

  closeModal();
  resetData();

  players.value = await (
    await getDocs(collection(db, "Demo"))
  ).docs.map((e) => e.data());
};

const resetData = () => {
  addData.name = "";
  addData.details = "";
  addData.overall = null;
  addData.surface = null;
  addData.hole = null;
};

let query: Ref<string | null> = ref("");
let searchPlayer: Ref<DocumentSnapshot<DocumentData> | null> = ref(null);

const search = async () => {
  if (!query.value?.trim()) {
    searchPlayer.value = null;
    query.value = null;
    return;
  }

  searchPlayer.value = await getDoc(
    doc(db, "Demo", query.value.toLowerCase().trim())
  );
};
</script>

<template>
  <div class="px-4 sm:px-6 lg:px-8 py-10">
    <div class="sm:flex sm:items-center">
      <div class="sm:flex-auto">
        <h1 class="text-xl font-semibold text-gray-900">Players</h1>
        <p class="mt-2 text-sm text-gray-700">
          A list of all the players I've remebered with their playstyles,
          teirlist rank, ratings, and more.
        </p>
      </div>
      <div class="mt-4 sm:mt-0 sm:ml-16 sm:flex-none">
        <button
          type="button"
          @click="openModal"
          class="inline-flex items-center justify-center rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 sm:w-auto"
        >
          Add user
        </button>
      </div>
    </div>
    <form class="sm:flex sm:items-center mt-8" @submit.prevent="search">
      <div class="sm:flex-auto">
        <input
          type="text"
          name="username"
          id="username"
          class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
          placeholder="Notch"
          v-model="query"
        />
      </div>
      <div class="mt-4 sm:mt-0 sm:ml-4 sm:flex-none">
        <button
          type="submit"
          class="inline-flex items-center justify-center rounded-md border border-transparent bg-gray-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 sm:w-auto"
        >
          Search
        </button>
      </div>
    </form>
    <div class="mt-8 flex flex-col">
      <div class="-my-2 -mx-4 overflow-x-auto sm:-mx-6 lg:-mx-8">
        <div class="inline-block min-w-full py-2 align-middle md:px-6 lg:px-8">
          <div
            class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 md:rounded-lg"
          >
            <table class="min-w-full divide-y divide-gray-300">
              <thead class="bg-gray-50">
                <tr>
                  <th
                    scope="col"
                    class="py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-6"
                  >
                    Name
                  </th>
                  <th
                    scope="col"
                    class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"
                  >
                    Playstyle
                  </th>
                  <th
                    scope="col"
                    class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"
                  >
                    Details
                  </th>
                  <th
                    scope="col"
                    class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"
                  >
                    Overall
                  </th>

                  <th
                    scope="col"
                    class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"
                  >
                    Surface
                  </th>

                  <th
                    scope="col"
                    class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"
                  >
                    Hole
                  </th>
                  <th
                    scope="col"
                    class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"
                  >
                    Rank
                  </th>
                  <th scope="col" class="relative py-3.5 pl-3 pr-4 sm:pr-6">
                    <span class="sr-only">Edit</span>
                  </th>
                </tr>
              </thead>
              <tbody
                class="divide-y divide-gray-200 bg-white"
                v-if="!searchPlayer"
              >
                <tr v-for="player in players" :key="player.username">
                  <td
                    class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6 flex gap-3"
                  >
                    <img
                      :src="`https://mc-heads.net/avatar/${player.username}`"
                      class="w-6 h-6"
                    />
                    {{ player.username }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ player.playstyle }}
                  </td>

                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ player.details }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ player.overall }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ player.surface }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ player.hole }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ player.rank || "Unranked" }}
                  </td>
                  <td
                    class="relative whitespace-nowrap py-4 pl-3 pr-4 text-right text-sm font-medium sm:pr-6"
                  >
                    <a href="#" class="text-indigo-600 hover:text-indigo-900"
                      >Edit<span class="sr-only"
                        >, {{ player.username }}</span
                      ></a
                    >
                  </td>
                </tr>
              </tbody>
              <tbody
                class="divide-y divide-gray-200 bg-white"
                v-else-if="!searchPlayer.exists()"
              >
                <tr>
                  <td
                    class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6 flex gap-3"
                  >
                    <img
                      src="https://mc-heads.net/avatar/steve"
                      class="w-6 h-6"
                    />
                    No user found
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    N/A
                  </td>

                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    N/A
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    N/A
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    N/A
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    N/A
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    N/A
                  </td>
                  <td
                    class="relative whitespace-nowrap py-4 pl-3 pr-4 text-right text-sm font-medium sm:pr-6"
                  >
                    <a href="#" class="text-indigo-600 hover:text-indigo-900"
                      >Edit<span class="sr-only">, N/A</span></a
                    >
                  </td>
                </tr>
              </tbody>
              <tbody class="divide-y divide-gray-200 bg-white" v-else>
                <tr>
                  <td
                    class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6 flex gap-3"
                  >
                    <img
                      :src="`https://mc-heads.net/avatar/${
                        searchPlayer.data().username
                      }`"
                      class="w-6 h-6"
                    />
                    {{ searchPlayer.data().username }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ searchPlayer.data().playstyle }}
                  </td>

                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ searchPlayer.data().details }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ searchPlayer.data().overall }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ searchPlayer.data().surface }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ searchPlayer.data().hole }}
                  </td>
                  <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">
                    {{ searchPlayer.data().rank || "Unranked" }}
                  </td>
                  <td
                    class="relative whitespace-nowrap py-4 pl-3 pr-4 text-right text-sm font-medium sm:pr-6"
                  >
                    <a href="#" class="text-indigo-600 hover:text-indigo-900"
                      >Edit<span class="sr-only"
                        >, {{ searchPlayer.data().username }}</span
                      ></a
                    >
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
  <TransitionRoot appear :show="isOpen" as="template">
    <Dialog as="div" @close="closeModal" class="relative z-10">
      <TransitionChild
        as="template"
        enter="duration-300 ease-out"
        enter-from="opacity-0"
        enter-to="opacity-100"
        leave="duration-200 ease-in"
        leave-from="opacity-100"
        leave-to="opacity-0"
      >
        <div class="fixed inset-0 bg-black bg-opacity-25" />
      </TransitionChild>

      <div class="fixed inset-0">
        <div
          class="flex min-h-full items-center justify-center p-4 text-center"
        >
          <TransitionChild
            as="template"
            enter="duration-300 ease-out"
            enter-from="opacity-0 scale-95"
            enter-to="opacity-100 scale-100"
            leave="duration-200 ease-in"
            leave-from="opacity-100 scale-100"
            leave-to="opacity-0 scale-95"
          >
            <DialogPanel
              class="w-full max-w-md transform rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all"
            >
              <DialogTitle
                as="h3"
                class="text-lg font-medium leading-6 text-gray-900"
              >
                Add a new player
              </DialogTitle>
              <div class="mt-2">
                <p class="text-sm text-gray-500">
                  Add a new player to the database. Tiers are not yet
                  implemented.
                </p>
                <!-- Username -->
                <div>
                  <label
                    for="username"
                    class="block text-sm font-medium text-gray-700 mt-2"
                    >Username</label
                  >
                  <div class="mt-1">
                    <input
                      type="text"
                      name="username"
                      id="username"
                      class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                      placeholder="Notch"
                      v-model="addData.name"
                    />
                  </div>
                </div>
                <!-- Playstyle -->
                <label
                  class="block text-sm font-medium text-gray-700 mt-2"
                  for=""
                  >Playstyle</label
                >
                <Listbox v-model="selectedPlaystyle">
                  <div class="relative mt-1">
                    <ListboxButton
                      class="relative w-full cursor-default rounded-lg bg-white py-2 pl-3 pr-10 text-left border focus:outline-none focus-visible:border-indigo-500 focus-visible:ring-2 focus-visible:ring-white focus-visible:ring-opacity-75 focus-visible:ring-offset-2 focus-visible:ring-offset-indigo-300 sm:text-sm border-gray-300 shadow-sm"
                    >
                      <span class="block truncate">{{
                        selectedPlaystyle.name
                      }}</span>
                      <span
                        class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-2"
                      >
                        <ChevronUpDownIcon
                          class="h-5 w-5 text-gray-400"
                          aria-hidden="true"
                        />
                      </span>
                    </ListboxButton>

                    <transition
                      leave-active-class="transition duration-100 ease-in"
                      leave-from-class="opacity-100"
                      leave-to-class="opacity-0"
                    >
                      <ListboxOptions
                        class="absolute mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none sm:text-sm"
                      >
                        <ListboxOption
                          v-slot="{ active, selected }"
                          v-for="playstyle in playstyles"
                          :key="playstyle.name"
                          :value="playstyle"
                          as="template"
                        >
                          <li
                            :class="[
                              active
                                ? 'bg-indigo-100 text-indigo-900'
                                : 'text-gray-900',
                              'relative cursor-default select-none py-2 pl-10 pr-4',
                            ]"
                          >
                            <span
                              :class="[
                                selected ? 'font-medium' : 'font-normal',
                                'block truncate',
                              ]"
                              >{{ playstyle.name }}</span
                            >
                            <span
                              v-if="selected"
                              class="absolute inset-y-0 left-0 flex items-center pl-3 text-indigo-600"
                            >
                              <CheckIcon class="h-5 w-5" aria-hidden="true" />
                            </span>
                          </li>
                        </ListboxOption>
                      </ListboxOptions>
                    </transition>
                  </div>
                </Listbox>
                <!-- Ratings -->
                <div class="grid grid-cols-3 gap-3">
                  <div class="flex flex-col">
                    <label
                      for="overall"
                      class="block text-sm font-medium text-gray-700 mt-2"
                      >Overall</label
                    >
                    <div class="mt-1">
                      <input
                        type="number"
                        name="overall"
                        id="overall"
                        class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                        placeholder="5"
                        step="0.5"
                        max="10"
                        min="0"
                        v-model="addData.overall"
                      />
                    </div>
                  </div>
                  <div class="flex flex-col">
                    <label
                      for="surface"
                      class="block text-sm font-medium text-gray-700 mt-2"
                      >Surface</label
                    >
                    <div class="mt-1">
                      <input
                        type="number"
                        name="surface"
                        id="surface"
                        class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                        placeholder="5"
                        step="0.5"
                        max="10"
                        min="0"
                        v-model="addData.surface"
                      />
                    </div>
                  </div>
                  <div class="flex flex-col">
                    <label
                      for="hole"
                      class="block text-sm font-medium text-gray-700 mt-2"
                      >Hole</label
                    >
                    <div class="mt-1">
                      <input
                        type="number"
                        name="hole"
                        id="hole"
                        class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                        placeholder="5"
                        step="0.5"
                        max="10"
                        min="0"
                        v-model="addData.hole"
                      />
                    </div>
                  </div>
                </div>
                <!-- Details -->
                <div>
                  <label
                    for="details"
                    class="block text-sm font-medium text-gray-700 mt-2"
                    >Details(Optional)</label
                  >
                  <div class="mt-1">
                    <input
                      type="text"
                      name="details"
                      id="details"
                      class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                      placeholder="Extra Details"
                      v-model="addData.details"
                    />
                  </div>
                </div>
              </div>

              <div class="mt-4">
                <button
                  type="button"
                  class="inline-flex justify-center rounded-md border border-transparent bg-indigo-100 px-4 py-2 text-sm font-medium text-indigo-900 hover:bg-indigo-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-500 focus-visible:ring-offset-2"
                  @click="submitUser"
                >
                  Add user
                </button>
              </div>
            </DialogPanel>
          </TransitionChild>
        </div>
      </div>
    </Dialog>
  </TransitionRoot>
</template>
