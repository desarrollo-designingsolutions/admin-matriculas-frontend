<script setup lang="ts">
import ModalFormDays from '@/pages/Course/Components/ModalFormDays.vue';

const props = defineProps({
    course_id: {
        type: Number,
        required: true,
    }
})

//TABLE
const refTableFull = ref()
const tableLoading = ref(false); // Estado de carga de la tabla

const optionsTable = {
    url: "/courseDay/list",
    paramsGlobal: {
        course_id: props.course_id,
    },
    headers: [
        { key: 'day_name', title: 'Dia', sortable: false },
        { key: 'time', title: 'Tiempo', sortable: false },
        { key: 'modalidad', title: 'Modalidad', sortable: false },
        { key: 'course_init', title: 'Iniciar curso', sortable: false },
        { key: 'price', title: 'Precio', sortable: false },
        { key: 'orden', title: 'Orden', sortable: false },
        { key: 'periodo', title: 'Período', sortable: false },
        { key: 'status', title: 'Estado', sortable: false },
        { key: 'actions', title: 'Acciones', sortable: false },
    ],
    actions: {
        view: {
            show: false,
        },
        delete: {
            url: "/courseDay/delete"
        },
    }
}

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
                    Dias
                </span>

                <div class="d-flex justify-end gap-3 flex-wrap ">
                    <VBtn @click="goCreate()">
                        Agregar dia
                    </VBtn>
                </div>
            </VCardTitle>

            <VCardText class="mt-2">
                <TableFull ref="refTableFull" :options="optionsTable" @edit="goEdit"
                    @update:loading="tableLoading = $event">
                </TableFull>
            </VCardText>
        </VCard>

        <ModalFormDays ref="refModalForm" :course_id="props.course_id" @execute="reloadTable" />
    </div>
</template>
