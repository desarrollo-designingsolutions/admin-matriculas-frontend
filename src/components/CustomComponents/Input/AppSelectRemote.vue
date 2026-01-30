<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';

const props = defineProps({
  url: {
    type: String,
    required: true
  },
  modelValue: {
    type: [Array, Object, String, Number],
    default: () => []
  },
  params: {
    type: [Object],
    default: () => { }
  },
  multiple: {
    type: Boolean,
    default: false
  },
  itemTitle: {
    type: String,
    default: 'title'
  },
  itemValue: {
    type: String,
    default: 'value'
  },
  searchParam: {
    type: String,
    default: 'searchQueryInfinite'
  },
  arrayInfo: {
    type: String,
    required: true,
  },
  disabled: {
    type: Boolean,
    required: false,
  },
  itemsData: {
    type: Array,
    required: false,
    default: () => [],
  },
  firstFetch: {
    type: Boolean,
    required: false,
    default: true,
  },
  // --- NUEVAS PROPS PARA EL BOTÓN DE AGREGAR ---
  showAddBtn: {
    type: Boolean,
    default: false
  },
  addBtnText: {
    type: String,
    default: 'Agregar nuevo'
  },
  addLayout: {
    type: String,
    default: 'icon-inner',
    validator: (value: string) => ['menu-top', 'icon-inner', 'button-append'].includes(value)
  }
});

// Agregamos 'click:add' a los eventos emitidos
const emit = defineEmits(['update:modelValue', 'click:add']);

const localValue = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
});

const items = ref<any[]>([]);
const search = ref('');
const page = ref(1);
const loading = ref(false);
const hasMore = ref(true);
const error = ref('');
const isSearching = ref(false);
const fetching = ref(false);
const paramsGlobal = ref({});

let searchTimeout: ReturnType<typeof setTimeout>;

const loadItems = async (newSearch = false) => {
  error.value = '';
  if (loading.value) return;

  loading.value = true;
  fetching.value = true;

  try {
    if (newSearch) {
      page.value = 1;
      items.value = [];
      hasMore.value = true;
      isSearching.value = !!search.value;
    }

    // Asegúrate de tener importado useAxios o la utilidad que uses
    const { data } = await useAxios(props.url).post({
      [props.searchParam]: search.value,
      page: page.value,
      ...paramsGlobal.value,
    });

    const arrayInfo = props.arrayInfo + "_arrayInfo";
    const countLinks = props.arrayInfo + "_countLinks";

    if (data.data) {
      items.value = newSearch ? data.data : [...items.value, ...data.data];
      hasMore.value = data.current_page < data.last_page;
    } else {
      items.value = newSearch ? data[arrayInfo] : [...items.value, ...data[arrayInfo]];
      hasMore.value = data[countLinks].length > 1;
    }

    page.value++;
  } catch (err) {
    error.value = 'Error al buscar datos.' + err;
  } finally {
    loading.value = false;
    fetching.value = false;
  }
};

watch(search, (newVal) => {
  clearTimeout(searchTimeout);

  if (!newVal && props.itemsData.length > 0 && isSearching.value) {
    isSearching.value = false;
    items.value = [...props.itemsData];
    return;
  }

  if (newVal) {
    searchTimeout = setTimeout(() => loadItems(true), 500);
  }
});

watch(() => props.itemsData, (newItemsData) => {
  if (newItemsData && newItemsData.length > 0 && !isSearching.value) {
    items.value = [...newItemsData];
  }
}, { deep: true });

onMounted(() => {
  if (props.itemsData.length > 0) {
    items.value = [...props.itemsData];
    return;
  }

  if (props.disabled) {
    return;
  }

  if (props.params) {
    paramsGlobal.value = props.params;
  }

  if (props.firstFetch) {
    loadItems();
  }
});

const clearText = () => {
  search.value = '';
  if (props.itemsData.length > 0) {
    isSearching.value = false;
    items.value = [...props.itemsData];
  } else if (isSearching.value) {
    isSearching.value = false;
    loadItems(true);
  }
}

const noDataMessage = computed(() => {
  if (loading.value) {
    return 'Buscando información...';
  }
  if (error.value) {
    return error.value;
  }

  return !search.value || search.value.trim() === ''
    ? 'Escribe algo para buscar...'
    : `No se encontraron resultados para "<strong>${search.value}</strong>".`;
});

const reloadItems = async (params = {}) => {
  items.value = [];
  paramsGlobal.value = params;
  loadItems();
}


// NUEVA FUNCIÓN: Agrega un item manualmente al array visual
const addNewItem = (newItem: any, rawTitleKey: string = 'name', rawValueKey: string = 'id') => {
  if (!newItem) return;

  // 1. Extraemos los valores del objeto nuevo usando las claves que nos pasen
  const titleContent = newItem[rawTitleKey]; // Ej: newItem.name
  const valueContent = newItem[rawValueKey]; // Ej: newItem.id

  // 2. Creamos un objeto híbrido. 
  // Mantenemos las propiedades originales (por si usas slots con 'color' u otros)
  // Pero FORZAMOS las claves 'title' y 'value' que es lo que el select espera por defecto.
  const formattedItem = {
    ...newItem,         // Mantenemos id, name, color, etc.
    title: titleContent, // Estandarizamos para el select
    value: valueContent  // Estandarizamos para el select
  };

  // 3. Verificamos duplicados basándonos en el valor
  // Usamos 'value' porque acabamos de crearlo
  const exists = items.value.some(i => i.value === formattedItem.value);

  if (!exists) {
    items.value.unshift(formattedItem);
    // Forzamos reactividad
    items.value = [...items.value];
  }
}

defineExpose({
  items,
  reloadItems,
  addNewItem
})
</script>

<template>
  <AppSelect @blur="clearText" returnObject v-model="localValue" :items="items" :item-title="itemTitle"
    :item-value="itemValue" :multiple="multiple" :search="search" :loading="fetching"
    @update:search="(value: string) => search = value" clearable :disabled="props.disabled">

    <template #prepend-item v-if="showAddBtn && addLayout === 'menu-top'">
      <VListItem class="pb-2" @click.stop @mousedown.stop @keydown.stop @keyup.stop>

        <AppTextField v-model="search" placeholder="Teclee para buscar..." variant="outlined" density="compact"
          hide-details clearable @click:clear="loadItems(true)" @click.stop @mousedown.stop @keydown.stop @keyup.stop
          @focus.stop @input.stop />

        <VBtn block color="primary" variant="tonal" size="small" class="mt-3" prepend-icon="tabler-plus"
          @click.stop="$emit('click:add')">
          {{ addBtnText }}
        </VBtn>

      </VListItem>
      <VDivider class="mb-2" />
    </template>

    <template #prepend-item v-else>
      <VListItem class="pb-2" @click.stop @mousedown.stop @keydown.stop @keyup.stop>
        <AppTextField v-model="search" placeholder="Teclee para buscar..." variant="outlined" density="compact"
          hide-details clearable @click:clear="loadItems(true)" @click.stop @mousedown.stop @keydown.stop @keyup.stop
          @focus.stop @input.stop />
      </VListItem>
      <VDivider class="mb-2" />
    </template>


    <template #append-inner v-if="showAddBtn && addLayout === 'icon-inner'">
      <VBtn icon variant="text" density="compact" color="primary" @click.stop="$emit('click:add')">
        <VIcon icon="tabler-plus" size="22" />
        <VTooltip activator="parent">{{ addBtnText }}</VTooltip>
      </VBtn>
    </template>


    <template #append v-if="showAddBtn && addLayout === 'button-append'">
      <VBtn color="primary" variant="tonal" class="rounded" icon="tabler-plus" @click.stop="$emit('click:add')">
        <VIcon icon="tabler-plus" />
        <VTooltip activator="parent">{{ addBtnText }}</VTooltip>
      </VBtn>
    </template>


    <template v-for="(_, name) in $slots" #[name]="slotProps">
      <slot v-if="!['prepend-item', 'append-inner', 'append'].includes(name as string)" :name="name"
        v-bind="slotProps || {}" />
    </template>

    <template v-slot:no-data>
      <VListItem>
        <VListItemTitle class="text-center" v-html="noDataMessage"></VListItemTitle>
      </VListItem>
    </template>

  </AppSelect>
</template>
