<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { router } from '@/plugins/1.router';
import moment from "moment";
import type { VForm } from 'vuetify/components/VForm';

definePage({
  path: "course-form/:action/:id?",
  name: "Course-Form",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "course.list",
  },
});

const { toast } = useToast()
const errorsBack = ref<IErrorsBack>({});
const disabledFiledsView = ref<boolean>(false);
const route = useRoute()
const formValidation = ref<VForm>()

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
  id: null as string | null,
  name: null as string | null,
  time: null as null | string,
  modalidad: null as null | string,
  course_init: null as null | string,
  price: null as null | string,
  orden: null as null | string,
  status: null as null | string,
  start_date: null as null | string,
  finish_date: null as null | string,
  discount: null as null | string,
  link: null as null | string,
  link_book: null as null | string,
  level_id: null as string | null,
  period_id: null as string | null,
  book_id: null as string | null,
  day_id: null as null | string,
})

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {
  form.value.id = route.params.id || null

  loading.form = true
  const { response, data } = await useAxios(`/course/${form.value.id}/edit`).get();

  if (response.status == 200 && data) {
    if (data.form) {
      form.value = dataClone(data.form);
    }
  }
  loading.form = false
}

const submitForm = async () => {
  const validation = await formValidation.value?.validate()
  if (validation?.valid) {

    const url = form.value.id ? `/course/update/${form.value.id}` : `/course/store`

    loading.form = true;
    const { data, response } = await useAxios(url).post(form.value);

    if (response.status == 200 && data) {

      if (data.code == 200) {
        router.push({ name: 'Course-List' })
      }
    }
    if (data.code === 422) errorsBack.value = data.errors ?? {};

    loading.form = false;
  }
  else {
    toast('Faltan Campos Por Diligenciar', '', 'danger')
  }
}

if (route.params.action == 'view') disabledFiledsView.value = true

onMounted(async () => {
  clearForm()
  if (route.params.id) {
    await fetchDataForm()
  }
})

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
</script>

<template>
  <div>
    <VCard :disabled="loading.form" :loading="loading.form">
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Fromulario Curso
        </span>
      </VCardTitle>

      <VCardText>
        <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
          <VRow>
            <VCol cols="12">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.name" label="Nombre"
                :error-messages="errorsBack.name" @input="errorsBack.name = ''" clearable />
            </VCol>

            <VCol cols="12" md="6">
              <AppSelectRemote :requiredField="true" label="Nivel" v-model="form.level_id" url="selectInfiniteLevel"
                arrayInfo="levels" :error-messages="errorsBack.level_id" @input="errorsBack.level_id = ''" clearable
                :loading="loading.countries" :rules="[requiredValidator]" />
            </VCol>

            <VCol cols="12" md="6">
              <AppSelectRemote :requiredField="true" label="Periodo" v-model="form.period_id" url="selectInfinitePeriod"
                arrayInfo="periods" :error-messages="errorsBack.period_id" @input="errorsBack.period_id = ''" clearable
                :loading="loading.countries" :rules="[requiredValidator]" />
            </VCol>

            <VCol cols="12" md="6">
              <AppSelectRemote label="Libro" v-model="form.book_id" url="selectInfiniteBook" arrayInfo="books"
                :error-messages="errorsBack.book_id" @input="errorsBack.book_id = ''" clearable />
            </VCol>

            <VCol cols="12" md="6">
              <AppSelectRemote :requiredField="true" label="Dia" v-model="form.day_id" url="selectInfiniteDay"
                arrayInfo="days" :error-messages="errorsBack.day_id" @input="errorsBack.day_id = ''" clearable
                :rules="[requiredValidator]" />
            </VCol>

            <VCol cols="12" md="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.time" label="Tiempo"
                :error-messages="errorsBack.time" @input="errorsBack.time = ''" clearable />
            </VCol>

            <VCol cols="12" md="4">
              <AppSelect :requiredField="true" :rules="[requiredValidator]" v-model="form.modalidad" label="Modalidad"
                :error-messages="errorsBack.modalidad" @input="errorsBack.modalidad = ''" clearable
                :items="modalitys" />
            </VCol>

            <VCol cols="12" md="4">
              <AppSelect :requiredField="true" :rules="[requiredValidator]" v-model="form.status" label="Estado"
                :error-messages="errorsBack.status" @input="errorsBack.status = ''" clearable :items="status" />
            </VCol>

            <VCol cols="12" md="6">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.course_init"
                label="Iniciar curso" :error-messages="errorsBack.course_init" @input="errorsBack.course_init = ''"
                clearable />
            </VCol>

            <VCol cols="12" md="6">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.orden" label="Orden"
                :error-messages="errorsBack.orden" @input="errorsBack.orden = ''" clearable type="number" />
            </VCol>

            <VCol cols="12" md="6">
              <AppDateTimePicker clearable :requiredField="true" label="Fecha Inicio" v-model="form.start_date"
                :error-messages="errorsBack.start_date" :rules="[requiredValidator]"
                :config="{ dateFormat: 'Y-m-d' }" />
            </VCol>

            <VCol cols="12" md="6">
              <AppDateTimePicker clearable :requiredField="true" :error-messages="errorsBack.finish_date"
                @input="errorsBack.finish_date = ''" v-model="form.finish_date" label="Fecha Final"
                @update:model-value="changeFinalDate($event)"
                :config="{ dateFormat: 'Y-m-d', disable: [{ from: `2020-01-01`, to: `${currentYear}-${currentMonth}-${currentDay}` }] }" />
            </VCol>

            <VCol cols="12" md="6">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.price" label="Precio"
                :error-messages="errorsBack.price" @input="errorsBack.price = ''" clearable type="number" />
            </VCol>

            <VCol cols="12" md="6">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.discount" label="Descuento"
                :error-messages="errorsBack.discount" @input="errorsBack.discount = ''" clearable type="number" />
            </VCol>

            <VCol cols="12" md="6">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.link"
                label="Link sin libro" :error-messages="errorsBack.link" @input="errorsBack.link = ''" clearable />
            </VCol>

            <VCol cols="12" md="6">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.link_book"
                label="Link con libro" :error-messages="errorsBack.link_book" @input="errorsBack.link_book = ''"
                clearable />
            </VCol>

            <!-- Botones de acción -->
            <VCol cols="12" class="d-flex justify-end gap-3 flex-wrap mt-5">
              <BtnBack :disabled="isLoading" :loading="isLoading" />
              <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
                color="primary">
                Guardar
              </VBtn>
            </VCol>
          </VRow>
        </VForm>
      </VCardText>
    </VCard>
  </div>
</template>
