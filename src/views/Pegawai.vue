<script setup>
import { 
        NCard, NDataTable, NButton, NSpace, NModal, NForm, 
        NFormItem, NInput, useMessage, useDialog 
    } 
from 'naive-ui';
import { h, ref, onMounted, computed } from 'vue';
import axios from 'axios'

const data = ref([])
const currentPage = ref(1)
const isLoading = ref(false)
const totalPage = ref(1)
const perPage = 10
const message = useMessage();
const dialog = useDialog()

const editId = ref(null)
const formData = ref({
  nama: '',
  usia: 0,
  jabatan: ''
})

const showModal = ref(false)

const deleteLoadingId = ref(null)

const onEditPegawai = (pegawai) => {
  editId.value = pegawai.id
  formData.value.nama = pegawai.nama
  formData.value.usia = pegawai.usia
  formData.value.jabatan = pegawai.jabatan
  showModal.value = true
}

const resetForm = () => {
  editId.value = null
  formData.value.nama = ''
  formData.value.usia = 0
  formData.value.jabatan = ''
  showModal.value = false
}

const onSubmitForm = () => {
  if (editId.value) {
    editPegawai()
  } else {
    insertPegawai()
  }
}

const onDeletePegawai = (pegawai) => {
    if (!deleteLoadingId.value) {
        dialog.warning({
            title: 'Confirm',
            content: 'Are you sure?',
            positiveText: 'Sure',
            negativeText: 'Not Sure',
            onPositiveClick: () => {
                message.info(
                    "Hapus Data Pegawai an: " +pegawai.nama+ "Berhasil",
                    {
                        keepAliveOnHover: true
                    }
                )
                deletePegawai(pegawai.id)
            },
            onNegativeClick: () => {
                message.error('Not Sure')
            }
        })
    }
}

const insertPegawai = () => {
  axios({
    method: 'post',
    baseURL: 'http://localhost:3000',
    url: '/pegawai',
    data: formData.value
  }).then((response) => {
    resetForm()
    getPegawai(1)
    message.info(
        "Tambah Data Pegawai Berhasil",
        {
            keepAliveOnHover: true
        }
    )
  }).catch((error) => {
    console.log(error.message)
  })
}

const editPegawai = () => {
  axios({
    method: 'patch',
    baseURL: 'http://localhost:3000',
    url: '/pegawai/' + editId.value,
    data: formData.value
  }).then((response) => {
    resetForm()
    getPegawai(currentPage.value)
    message.info(
        "Edit Data Pegawai Berhasil",
        {
            keepAliveOnHover: true
        }
    )
  }).catch((error) => {
    console.log(error.message)
  })
}

const deletePegawai = (id) => {
  deleteLoadingId.value = id
  axios({
    method: 'delete',
    baseURL: 'http://localhost:3000',
    url: '/pegawai/' + id
  }).then((response) => {
    getPegawai(currentPage.value)
    message.info(
        "Hapus Data Pegawai Berhasil",
        {
            keepAliveOnHover: true
        }
    )
  }).catch((error) => {
    console.log(error.message)
  }).finally(() => {
    deleteLoadingId.value = null
  })
}

const getPegawai = (page) => {
  currentPage.value = page
  isLoading.value = true
  axios({
    method: 'get',
    baseURL: 'http://localhost:3000',
    url: '/pegawai',
    params: {
      _page: page,
      _limit: perPage,
      _sort: 'id',
      _order: 'asc'
    }
  }).then((response) => {
    totalPage.value = Math.ceil(response.headers['x-total-count'] / perPage)

    console.log(response.db)


    data.value = response.data
  }).catch((error) => {
    console.log(error.message)
  }).finally(() => {
    isLoading.value = false
  })
}

onMounted(() => {
  getPegawai(1)
})

const columns = [
  { title: "No", key: 'id', size: 2 },
  { title: "Nama", key: 'nama' },
  { title: "Usia", key: 'usia' },
  { title: "Jabatan", key: 'jabatan' },
  {
    title: "Action",
    key: 'action',
    render: (row) => {
      return h(NSpace, {}, [
        h(
          NButton,
          {
            type: 'info',
            ghost: true,
            onClick: () => onEditPegawai(row)
          },
          { default: () => 'Edit' }
        ),
        h(
          NButton,
          {
            type: 'error',
            ghost: true,
            onClick: () => onDeletePegawai(row),
            loading: row.id == deleteLoadingId.value
          },
          { default: () => 'Delete' }
        )
      ])
    }
  },
]

const pagination = computed(() => {
  return {
    pageSize: perPage,
    pageCount: totalPage.value,
    page: currentPage.value
  }
})

const handlePageChange = (page) => {
  getPegawai(page)
}
</script>

<template>
  <NCard title="Pegawai">
    <NButton @click="showModal = true">Add New</NButton>
    <NDataTable 
      remote
      :loading="isLoading"
      :data="data"
      :columns="columns"
      :pagination="pagination"
      @update:page="handlePageChange" />
  </NCard>
  <NModal v-model:show="showModal">
    <NCard style="width: 50%" title="Data Pegawai">
      <NForm
        @submit.prevent="onSubmitForm"
        :model="formData"
        label-placement="left"
        label-width="auto"
        require-mark-placement="right-hanging"
        size="medium">
        <NFormItem label="Nama" path="nama">
          <NInput v-model:value="formData.nama" />
        </NFormItem>
        <NFormItem label="Usia" path="usia">
          <NInput v-model:value.number="formData.usia" />
        </NFormItem>
        <NFormItem label="Jabatan" path="jabatan">
          <NInput v-model:value="formData.jabatan" />
        </NFormItem>
        <NFormItem label="&nbsp;">
          <NButton type="info" ghost attr-type="submits">Save</NButton>
        </NFormItem>
      </NForm>
    </NCard>
  </NModal>
</template>
