<script setup lang="ts">
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import moment from "moment";
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
  start_date: null as null | string,
  finish_date: null as null | string,
  discount: null as null | string,
  payback: null as null | string,
  link: null as null | string,
  link_book: null as null | string,
  day_course_id: null as null | string,
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

  titleModal.value = id ? "Editar Descuento" : "Crear Descuento"

  form.value.id = id

  if (form.value.id) {
    await fetchDataForm();
  }
};

const fetchDataForm = async () => {
  const url = form.value.id ? `/discount/${form.value.id}/edit` : `/discount/create`

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
    const url = form.value.id ? `/discount/update/${form.value.id}` : `/discount/store`

    loading.form = true

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

//OBTENGO EL DIA DE HOY PARA CALCULAR LA FECHA INICIO
const now = new Date();
const currentYear = now.getFullYear();
const currentMonth = now.getMonth() + 1; // Los meses van de 0 a 11, por lo que sumamos 1
const currentDay = now.getDate();

const changeFinalDate = (event: any) => {
  if (event) {
    let d1 = moment(`${currentYear}-${currentMonth.toString().padStart(2, '0')}-${currentDay.toString().padStart(2, '0')}`);
    let d2 = moment(event);
    errorsBack.value.final_date = "";
    if (!d2.isAfter(d1))
      errorsBack.value.final_date = `La fecha debe ser posterior a ${d1.format('YYYY-MM-DD')}`;
  }
}

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
                <AppSelectRemote :requiredField="true" label="Dia del curso" v-model="form.day_course_id"
                  url="selectInfiniteCourseDay" arrayInfo="courseDays" :error-messages="errorsBack.day_course_id"
                  @input="errorsBack.day_course_id = ''" clearable :rules="[requiredValidator]" :params="{
                    course_id: props.course_id,
                  }" />
              </VCol>

              <VCol cols="12">
                <AppDateTimePicker clearable :requiredField="true" label="Fecha Inicio" v-model="form.start_date"
                  :error-messages="errorsBack.start_date" :rules="[requiredValidator]"
                  :config="{ dateFormat: 'Y-m-d' }" />
              </VCol>

              <VCol cols="12">
                <AppDateTimePicker clearable :requiredField="true" :error-messages="errorsBack.finish_date"
                  @input="errorsBack.finish_date = ''" v-model="form.finish_date" label="Fecha Final"
                  @update:model-value="changeFinalDate($event)"
                  :config="{ dateFormat: 'Y-m-d', disable: [{ from: `2020-01-01`, to: `${currentYear}-${currentMonth}-${currentDay}` }] }" />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.discount"
                  label="Descuento" :error-messages="errorsBack.discount" @input="errorsBack.discount = ''" clearable
                  type="number" />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.payback" label="Payback"
                  :error-messages="errorsBack.payback" @input="errorsBack.payback = ''" clearable type="number" />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.link"
                  label="Link sin libro" :error-messages="errorsBack.link" @input="errorsBack.link = ''" clearable />
              </VCol>

              <VCol cols="12">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.link_book"
                  label="Link con libro" :error-messages="errorsBack.link_book" @input="errorsBack.link_book = ''"
                  clearable />
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
