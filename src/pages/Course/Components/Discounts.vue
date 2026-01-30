<script setup lang="ts">
import ModalFormDiscount from '@/pages/Course/Components/ModalFormDiscount.vue';

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
    url: "/discount/list",
    paramsGlobal: {
        course_id: props.course_id,
    },
    headers: [
        { key: 'day_course_name', title: 'Dia', sortable: false },
        { key: 'start_date', title: 'Fecha inicio', sortable: false },
        { key: 'finish_date', title: 'Fecha fin', sortable: false },
        { key: 'discount', title: 'Descuento', sortable: false },
        { key: 'payback', title: 'Payback', sortable: false },
        { key: 'actions', title: 'Acciones', sortable: false },
    ],
    actions: {
        view: {
            show: false,
        },
        delete: {
            url: "/discount/delete"
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
                    Descuentos
                </span>

                <div class="d-flex justify-end gap-3 flex-wrap ">
                    <VBtn @click="goCreate()">
                        Agregar descuento
                    </VBtn>
                </div>
            </VCardTitle>

            <VCardText class="mt-2">
                <TableFull ref="refTableFull" :options="optionsTable" @edit="goEdit"
                    @update:loading="tableLoading = $event">
                </TableFull>
            </VCardText>
        </VCard>

        <ModalFormDiscount ref="refModalForm" :course_id="props.course_id" @execute="reloadTable" />
    </div>
</template>
