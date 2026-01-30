<script setup lang="ts">
import ModalFormBook from '@/pages/Course/Components/ModalFormBook.vue';

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
    url: "/courseBook/list",
    paramsGlobal: {
        course_id: props.course_id,
    },
    headers: [
        { key: 'book_name', title: 'Libro', sortable: false },
        { key: 'actions', title: 'Acciones', sortable: false },
    ],
    actions: {
        view: {
            show: false,
        },
        delete: {
            url: "/courseBook/delete"
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
                    Libros
                </span>

                <div class="d-flex justify-end gap-3 flex-wrap ">
                    <VBtn @click="goCreate()">
                        Agregar Libro
                    </VBtn>
                </div>
            </VCardTitle>

            <VCardText class="mt-2">
                <TableFull ref="refTableFull" :options="optionsTable" @edit="goEdit"
                    @update:loading="tableLoading = $event">
                </TableFull>
            </VCardText>
        </VCard>

        <ModalFormBook ref="refModalForm" :course_id="props.course_id" @execute="reloadTable" />
    </div>
</template>
