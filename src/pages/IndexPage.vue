<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleAddData">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <q-input
                v-if="props.row.isEditing"
                v-model="props.row[col.name]"
                dense
                borderless
                @keyup.enter="handleUpdate(props.row)"
              />
              <div v-else>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface RowData {
  id: string;
  name: string;
  age: number;
  isEditing: boolean;
}

interface ApiResponse {
  id: string;
  name: string;
  age: number;
}

const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

async function fetchData(): Promise<void> {
  try {
    const response = await axios.get<ApiResponse[]>('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value = response.data.map((item) => ({ ...item, isEditing: false } as RowData));
    console.log(blockData.value);
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

onMounted(async () => {
  await fetchData();
});

async function handleClickOption(btn: btnType, data: RowData) {
  if (btn.status === 'delete') {
    try {
      await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`);
      console.log('Data deleted successfully');
      await fetchData();
    } catch (error) {
      console.error('Error deleting data:', error);
    }
  } else if (btn.status === 'edit') {
    if (data.isEditing) {
      try {
        const response = await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', {
          name: data.name,
          age: parseInt(data.age.toString()),
          id: data.id
        });
        console.log('Data updated successfully:', response.data);
        data.isEditing = false;
        await fetchData();
      } catch (error) {
        console.error('Error updating data:', error);
      }
    } else {
      // Toggle editing mode
      data.isEditing = true;
    }
  }
}

async function handleUpdate(row: RowData): Promise<void> {
  try {
    const response = await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', {
      name: row.name,
      age: row.age, 
      id: row.id
    });
    console.log('Data updated successfully:', response.data);
    row.isEditing = false;
    await fetchData();
  } catch (error) {
    console.error('Error updating data:', error);
  }
}

async function handleAddData() {
  try {
    const response = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
      name: tempData.value.name,
      age: parseInt(tempData.value.age)
    });
    console.log('Data added successfully:', response.data);
    await fetchData();
    tempData.value = { name: '', age: '' };
  } catch (error) {
    console.error('Error adding data:', error);
  }
}

</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
