<template>
  <div class="space-y-6 sm:px-6 lg:px-0 w-full">
    <div class="bg-white mx-auto w-full">
      <div class="px-10 py-12 flex items-center text-2xl font-medium gap-2">
        <h1>{{ page.name }}</h1>
        <ChevronRightIcon class="inline text-gray-500 h-6 w-auto" />
        <h1>Item</h1>
      </div>
      <div class="flex flex-col gap-6">
        <!-- Attribute Inputs -->
        <div v-for="attribute in page.attributes" :key="attribute.id">
          <div class="px-10">
            <label :for="attribute.id" class="block text-sm font-medium text-gray-700">{{ attribute.label }} <span v-if="attribute.required" class="text-gray-400 font-normal pl-2">required</span></label>

            <!-- If input is read-only -->
            <textarea v-if="(page.readonly || attribute.readonly) && attribute.type === AttributeType.LongText" readonly :id="attribute.id" :value="items[0][attribute.id]" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-0 focus:border-gray-300 sm:text-sm" />
            <input v-else-if="page.readonly || attribute.readonly" type="text" readonly :id="attribute.id" :value="items[0][attribute.id]" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-0 focus:border-gray-300 sm:text-sm" />

            <!-- Else input is writeable -->
            <input v-else-if="attribute.type === AttributeType.Date" type="date" :disabled="loading" :id="attribute.id" :value="items[0][attribute.id]" @input="update(attribute.id, ($event.target as HTMLInputElement).value)" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-gray-900 focus:border-gray-900 sm:text-sm" />
            <select v-else-if="attribute.type === AttributeType.Bool" :disabled="loading" :id="attribute.id" :value="items[0][attribute.id]" @input="update(attribute.id, ($event.target as HTMLInputElement).value)" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-gray-900 focus:border-gray-900 sm:text-sm">
              <option :value="true">true</option>
              <option :value="false">false</option>
            </select>
            <select v-else-if="attribute.type === AttributeType.Enum" :disabled="loading" :id="attribute.id" :value="items[0][attribute.id]" @input="update(attribute.id, ($event.target as HTMLInputElement).value)" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-gray-900 focus:border-gray-900 sm:text-sm">
              <option v-for="option in attribute.enumOptions" :key="option" :value="option">{{ option }}</option>
            </select>
            <textarea v-else-if="attribute.type === AttributeType.LongText" :disabled="loading" :id="attribute.id" :value="items[0][attribute.id]" @input="update(attribute.id, ($event.target as HTMLInputElement).value)" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-gray-900 focus:border-gray-900 sm:text-sm" />
            <input v-else type="text" :disabled="loading" :id="attribute.id" :value="items[0][attribute.id]" @input="update(attribute.id, ($event.target as HTMLInputElement).value)" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-gray-900 focus:border-gray-900 sm:text-sm" />
          </div>
        </div>
        <!-- Warning -->
        <div v-if="warning" class="px-4 md:px-10 text-sm text-red-500">
          {{ warning }}
        </div>
        <!-- Buttons -->
        <div class="px-10 flex justify-end gap-4">
          <button class="bg-white border border-transparent rounded-md py-2 px-4 inline-flex justify-center text-sm font-medium text-gray-800 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
            @click="router.go(-1)">
            Back
          </button>
          <button v-if="!page.readonly" :disabled="!haveUnsavedChanges || loading" class="bg-green-500 border border-transparent rounded-md shadow-sm py-2 px-4 inline-flex justify-center text-sm font-medium text-white hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 disabled:bg-gray-300 disabled:hover:bg-gray-300 disabled:focus:ring-gray-300"
            @click="upsertItem(itemId)">
            {{ loading ? 'Loading...' : 'Save' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { PropType } from 'vue'
import { ChevronRightIcon } from '@heroicons/vue/solid'
import router from '../../router'
import { initLoading, initCrud } from '../../utils/dashboard'
import { AttributeType, Page } from '../../utils/config'

const props = defineProps({
  loading: {
    type: Boolean,
    default: false,
  },
  page: {
    type: Object as PropType<Page>,
    required: true,
  },
  itemId: {
    type: String,
    default: '',
  },
})

const { loading } = initLoading(props.loading)
const { page, warning, haveUnsavedChanges, items, getItem, upsertItem } = initCrud(loading, props.page)

function update (attributeId:string, newVal:string) {
  items.value[0][attributeId] = newVal
  haveUnsavedChanges.value = true
}

getItem(props.itemId)
</script>
