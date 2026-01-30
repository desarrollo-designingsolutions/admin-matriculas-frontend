<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { router } from '@/plugins/1.router';
import type { VForm } from 'vuetify/components/VForm';
import Books from './Components/Books.vue';
import Days from './Components/Days.vue';
import Discounts from './Components/Discounts.vue';

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
const currentTab = ref(0)

const loading = reactive({
  form: false,
})

const form = ref({
  id: null as string | null,
  name: null as string | null,
  level_id: null as string | null,
  period_id: null as string | null,
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
        <VTabs v-model="currentTab">
          <VTab>Datos</VTab>
          <VTab :disabled="!form.id">Libros</VTab>
          <VTab :disabled="!form.id">Días</VTab>
          <VTab :disabled="!form.id">Descuentos</VTab>
        </VTabs>
      </VCardText>

      <VCardText>
        <VWindow v-model="currentTab">
          <VWindowItem :key="0">
            <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
              <VRow>
                <VCol cols="12">
                  <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.name" label="Nombre"
                    :error-messages="errorsBack.name" @input="errorsBack.name = ''" clearable />
                </VCol>

                <VCol cols="12">
                  <AppSelectRemote :requiredField="true" label="Nivel" v-model="form.level_id" url="selectInfiniteLevel"
                    arrayInfo="levels" :error-messages="errorsBack.level_id" @input="errorsBack.level_id = ''" clearable
                    :loading="loading.countries" :rules="[requiredValidator]" />
                </VCol>

                <VCol cols="12">
                  <AppSelectRemote :requiredField="true" label="Periodo" v-model="form.period_id"
                    url="selectInfinitePeriod" arrayInfo="periods" :error-messages="errorsBack.period_id"
                    @input="errorsBack.period_id = ''" clearable :loading="loading.countries"
                    :rules="[requiredValidator]" />
                </VCol>

                <VCol cols="12" class="d-flex justify-end gap-3 flex-wrap mt-5">
                  <BtnBack :disabled="isLoading" :loading="isLoading" />
                  <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
                    color="primary">
                    Guardar
                  </VBtn>
                </VCol>

              </VRow>
            </VForm>
          </VWindowItem>

          <VWindowItem :key="1">
            <Books :course_id="form.id" />
          </VWindowItem>

          <VWindowItem :key="2">
            <Days :course_id="form.id" />
          </VWindowItem>

          <VWindowItem :key="3">
            <Discounts :course_id="form.id" />
          </VWindowItem>
        </VWindow>
      </VCardText>
    </VCard>
  </div>
</template>
