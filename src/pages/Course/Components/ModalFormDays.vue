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
const modalitys = [
  {
    'title': 'Presencial',
    'value': 'Presencial',
  },
  {
    'title': 'Virtual',
    'value': 'Virtual',
  },
]
const status = [
  {
    'title': 'Activo',
    'value': 'Activo',
  },
  {
    'title': 'Finalizado',
    'value': 'Finalizado',
  },
]

const loading = reactive({
  form: false,
})

const form = ref({
  id: null as null | string,
  time: null as null | string,
  modalidad: null as null | string,
  course_init: null as null | string,
  price: null as null | string,
  orden: null as null | string,
  periodo: null as null | string,
  status: null as null | string,
  course_id: null as null | string,
  day_id: null as null | string,
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

  titleModal.value = id ? "Editar Dia" : "Crear Dia"

  form.value.id = id

  if (form.value.id) {
    await fetchDataForm();
  }
};

const fetchDataForm = async () => {
  const url = form.value.id ? `/courseDay/${form.value.id}/edit` : `/courseDay/create`

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
    const url = form.value.id ? `/courseDay/update/${form.value.id}` : `/courseDay/store`

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
                <AppSelectRemote :requiredField="true" label="Dia" v-model="form.day_id" url="selectInfiniteDay"
                  arrayInfo="days" :error-messages="errorsBack.day_id" @input="errorsBack.day_id = ''" clearable
                  :rules="[requiredValidator]" />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.time" label="Tiempo"
                  :error-messages="errorsBack.time" @input="errorsBack.time = ''" clearable />
              </VCol>

              <VCol cols="12">
                <AppSelect :requiredField="true" :rules="[requiredValidator]" v-model="form.modalidad" label="Modalidad"
                  :error-messages="errorsBack.modalidad" @input="errorsBack.modalidad = ''" clearable
                  :items="modalitys" />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.course_init"
                  label="Iniciar curso" :error-messages="errorsBack.course_init" @input="errorsBack.course_init = ''"
                  clearable />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.price" label="Precio"
                  :error-messages="errorsBack.price" @input="errorsBack.price = ''" clearable type="number" />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.orden" label="Orden"
                  :error-messages="errorsBack.orden" @input="errorsBack.orden = ''" clearable type="number" />
              </VCol>

              <VCol cols="12">
                <AppSelectRemote :requiredField="true" label="Periodo" v-model="form.periodo" url="selectInfinitePeriod"
                  arrayInfo="periods" :error-messages="errorsBack.periodo" @input="errorsBack.periodo = ''" clearable
                  :rules="[requiredValidator]" />
              </VCol>

              <VCol cols="12">
                <AppSelect :requiredField="true" :rules="[requiredValidator]" v-model="form.status" label="Estado"
                  :error-messages="errorsBack.status" @input="errorsBack.status = ''" clearable :items="status" />
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
