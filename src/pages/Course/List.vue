<script setup lang="ts">
import { router } from '@/plugins/1.router';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

definePage({
  name: "Course-List",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "course.list",
  },
});

const authenticationStore = useAuthenticationStore();

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
const optionsTable = {
  url: "/course/list",
  headers: [
    { key: 'name', title: 'Nombre' },
    { key: 'level_name', title: 'Nivel' },
    { key: 'period_name', title: 'Periodo' },
    { key: 'actions', title: 'Acciones', sortable: false },
  ],
  actions: {
    view: {
      show: false,
    },
    delete: {
      url: "/course/delete"
    },
  }
}

const goViewEdit = (data: any) => {
  router.push({ name: "Course-Form", params: { action: "edit", id: data.id } })
}

const goViewCreate = () => {
  router.push({ name: "Course-Form", params: { action: "create" } })
}



const selectCompany = (company: object) => {
  authenticationStore.company = company;
  router.push({ name: "Home" });
};


const tableLoading = ref(false); // Estado de carga de la tabla

// Método para refrescar los datos
const refreshTable = () => {
  if (refTableFull.value) {
    refTableFull.value.fetchTableData(null, false, true); // Forzamos la búsqueda
  }
};

</script>

<template>
  <div>

    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Cursos
        </span>

        <div class="d-flex justify-end gap-3 flex-wrap ">
          <VBtn @click="goViewCreate()">
            Agregar curso
          </VBtn>
        </div>
      </VCardTitle>

      <!-- Sección de filtros mejorada -->
      <VCardText>
        <FilterDialog :options-filter="optionsFilter" @force-search="refreshTable" :table-loading="tableLoading">
        </FilterDialog>
      </VCardText>

      <VCardText class="mt-2">
        <TableFull ref="refTableFull" :options="optionsTable" @edit="goViewEdit"
          @update:loading="tableLoading = $event">
        </TableFull>
      </VCardText>
    </VCard>
  </div>
</template>
