<script setup lang="ts">
import ModalForm from '@/pages/Level/Components/ModalForm.vue';

definePage({
  name: "Level-List",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "level.list",
  },
});

//FILTER
const optionsFilter = ref({
  dialog: {
    cols: "12",
    width: 500,
    inputs: [],
  },
})

//TABLE
const refTableFull = ref()
const tableLoading = ref(false); // Estado de carga de la tabla

const optionsTable = {
  url: "/level/list",
  headers: [
    { key: 'id', title: 'ID' },
    { key: 'level', title: 'Nivel' },
    { key: 'actions', title: 'Acciones', sortable: false },
  ],
  actions: {
    view: {
      show: false,
    },
    delete: {
      url: "/level/delete"
    },
  }
}

// Método para refrescar los datos
const refreshTable = () => {
  if (refTableFull.value) {
    refTableFull.value.fetchTableData(null, false, true); // Forzamos la búsqueda
  }
};

//ModalForm
const refModalForm = ref()

const reloadTable = () => {
  refTableFull.value.fetchTableData()
}

const goEdit = (data: any) => {
  refModalForm.value.openModal(data.id, false)
}

const goCreate = () => {
  refModalForm.value.openModal()
}

</script>

<template>
  <div>
    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Nivel
        </span>

        <div class="d-flex justify-end gap-3 flex-wrap ">
          <VBtn @click="goCreate()">
            Agregar Nivel
          </VBtn>
        </div>
      </VCardTitle>

      <!-- Sección de filtros mejorada -->
      <VCardText>
        <FilterDialog :options-filter="optionsFilter" @force-search="refreshTable" :table-loading="tableLoading">
        </FilterDialog>
      </VCardText>

      <VCardText class="mt-2">

        <TableFull ref="refTableFull" :options="optionsTable" @edit="goEdit" @update:loading="tableLoading = $event">
        </TableFull>
      </VCardText>
    </VCard>

    <ModalForm ref="refModalForm" @execute="reloadTable" />
  </div>
</template>
