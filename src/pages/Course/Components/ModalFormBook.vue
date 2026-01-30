<script setup lang="ts">
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import type { VForm } from "vuetify/components/VForm";

const props = defineProps({
  course_id: {
    type: Number,
    required: true,
  }
})

const errorsBack = ref<IErrorsBack>({});
const refForm = ref<VForm>();
const emit = defineEmits(["execute"])

const titleModal = ref<string>("")
const isDialogVisible = ref<boolean>(false)
const disabledFiledsView = ref<boolean>(false)

const loading = reactive({
  form: false,
})

const form = ref({
  id: null as null | string,
  book_id: null as null | string,
  course_id: null as null | string,
});

const handleClearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const handleDialogVisible = () => {
  isDialogVisible.value = !isDialogVisible.value;
};

const openModal = async (id: string | null = null, disabled: boolean = false) => {
  disabledFiledsView.value = disabled

  handleClearForm()
  handleDialogVisible();

  titleModal.value = id ? "Editar Libro" : "Crear Libro"

  form.value.id = id

  if (form.value.id) {
    await fetchDataForm();
  }
};

const fetchDataForm = async () => {

  const url = form.value.id ? `/courseBook/${form.value.id}/edit` : `/courseBook/create`

  loading.form = true
  const { data, response } = await useAxios(url).get();
  loading.form = false

  if (response.status == 200 && data) {
    if (data.form) {
      form.value = dataClone(data.form);
    }
  }
}

const submitForm = async () => {
  const validation = await refForm.value?.validate();

  if (validation?.valid) {
    const url = form.value.id ? `/courseBook/update/${form.value.id}` : `/courseBook/store`

    loading.form = true

    form.value.course_id = props.course_id
    const { response, data } = await useAxios(url).post(form.value);

    if (response.status == 200 && data) {
      emit("execute", data.data)
      handleDialogVisible()
    }

    if (data.code == 422) {
      errorsBack.value = data.errors ?? {}
    }

    loading.form = false
  }
}

// Computed que verifica si al menos uno de los valores es true
const isLoading = computed(() => {
  return Object.values(loading).some(value => value);
});

defineExpose({
  openModal
})

</script>

<template>
  <div>
    <VDialog v-model="isDialogVisible" :overlay="false" max-width="40rem" transition="dialog-transition" persistent>
      <DialogCloseBtn @click="handleDialogVisible" />
      <VCard :loading="isLoading" :disabled="isLoading" class="w-100">
        <!-- Toolbar -->
        <div>
          <VToolbar color="primary">
            <VToolbarTitle>{{ titleModal }}</VToolbarTitle>
          </VToolbar>
        </div>

        <VCardText>
          <VForm ref="refForm" @submit.prevent="() => { }">
            <VRow>
              <VCol cols="12">
                <AppSelectRemote :requiredField="true" label="Libro" v-model="form.book_id" url="selectInfiniteBook"
                  arrayInfo="books" :error-messages="errorsBack.book_id" @input="errorsBack.book_id = ''" clearable
                  :disabled="disabledFiledsView" :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>
        </VCardText>

        <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
          <VBtn color="secondary" :disabled="isLoading" :loading="isLoading" @click="handleDialogVisible">Cancelar
          </VBtn>
          <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
            color="primary">
            Guardar
          </VBtn>
        </VCardText>
      </VCard>
    </VDialog>
  </div>
</template>
