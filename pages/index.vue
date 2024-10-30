
<template>
  <div>
    <div class="card">
      <Toolbar class="mb-6">
        <template #start>
          <Button label="New" icon="pi pi-plus" class="mr-2" @click="openNew" />
          <Button label="Delete" icon="pi pi-trash" severity="danger" outlined @click="confirmDeleteSelected" :disabled="!selectedCharacters || !selectedCharacters.length" />
        </template>
      </Toolbar>

      <DataTable
        ref="dt"
        v-model:selection="selectedCharacters"
        :value="characters"
        dataKey="name"
        :paginator="true"
        :rows="20"
        :filters="filters"
        paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
        :rowsPerPageOptions="[5, 10, 20, 30]"
        currentPageReportTemplate="Showing {first} to {last} of {totalRecords} characters"
      >
        <template #header>
          <div class="flex flex-wrap gap-2 items-center justify-between">
            <h4 class="m-0">Manage Characters</h4>
            <IconField>
              <InputIcon><i class="pi pi-search" /></InputIcon>
              <InputText v-model="filters['global'].value" placeholder="Search..." />
            </IconField>
          </div>
        </template>

        <Column selectionMode="multiple" style="width: 3rem" :exportable="false"></Column>
        <Column field="name" header="Name" sortable style="min-width: 16rem"></Column>
        <Column field="race" header="Race" sortable style="min-width: 16rem"></Column>
        <Column field="className" header="Class" sortable style="min-width: 16rem"></Column>
        <Column field="level" header="Level" sortable style="min-width: 16rem"></Column>
        <Column field="createdAt" header="Created at" sortable style="min-width: 16rem"></Column>

        <Column :exportable="false" style="min-width: 12rem">
          <template #body="slotProps">
            <Button icon="pi pi-pencil" outlined rounded class="mr-2" @click="editCharacter(slotProps.data)" />
            <Button icon="pi pi-trash" outlined rounded severity="danger" @click="confirmDeleteCharacter(slotProps.data)" />
          </template>
        </Column>
      </DataTable>
    </div>

    <Dialog v-model:visible="characterDialog" :style="{ width: '450px' }" header="Character Details" :modal="true">
      <div class="flex flex-col gap-6">
        <div>
          <label for="name" class="block font-bold mb-3">Name</label>
          <InputText id="name" v-model.trim="character.name" required="true" :invalid="submitted && !character.name" fluid />
          <small v-if="submitted && !character.name" class="text-red-500">Name is required.</small>
        </div>
        <div>
          <label for="race" class="block font-bold mb-3">Race</label>
          <InputText id="race" v-model.trim="character.race" required="true" :invalid="submitted && !character.race" fluid />
          <small v-if="submitted && !character.race" class="text-red-500">Race is required.</small>
        </div>
        <div>
          <label for="className" class="block font-bold mb-3">Class</label>
          <InputText id="className" v-model.trim="character.className" required="true" :invalid="submitted && !character.className" fluid />
          <small v-if="submitted && !character.className" class="text-red-500">Class is required.</small>
        </div>
        <div>
          <label for="level" class="block font-bold mb-3">Level</label>
          <InputNumber id="level" v-model.trim="character.level" required="true" :invalid="submitted && !character.level" fluid />
          <small v-if="submitted && !character.level" class="text-red-500">Level is required.</small>
        </div>
      </div>

      <template #footer>
        <Button label="Cancel" icon="pi pi-times" text @click="hideDialog" />
        <Button label="Save" icon="pi pi-check" @click="saveCharacter" />
      </template>
    </Dialog>

    <Dialog v-model:visible="deleteCharacterDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
      <div class="flex items-center gap-4">
        <i class="pi pi-exclamation-triangle !text-3xl" />
        <span v-if="character">Are you sure you want to delete <b>{{ character.name }}</b>?</span>
      </div>
      <template #footer>
        <Button label="No" icon="pi pi-times" text @click="deleteCharacterDialog = false" />
        <Button label="Yes" icon="pi pi-check" @click="deleteCharacter" />
      </template>
    </Dialog>

    <Dialog v-model:visible="deleteCharactersDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
      <div class="flex items-center gap-4">
        <i class="pi pi-exclamation-triangle !text-3xl" />
        <span v-if="character">Are you sure you want to delete the selected characters?</span>
      </div>
      <template #footer>
        <Button label="No" icon="pi pi-times" text @click="deleteCharacterDialog = false" />
        <Button label="Yes" icon="pi pi-check" text @click="deleteSelectedCharacters" />
      </template>
    </Dialog>
  </div>
</template>

<script setup>
import 'primeicons/primeicons.css';

import { ref, onMounted } from 'vue';
import { FilterMatchMode } from '@primevue/core/api';
import { useToast } from 'primevue/usetoast';

const dt = ref();
const filters = ref({ 'global': { value: null, matchMode: FilterMatchMode.CONTAINS } });
const toast = useToast();
const characterDialog = ref(false);
const deleteCharacterDialog = ref(false);
const deleteCharactersDialog = ref(false);

const submitted = ref(false);
const selectedCharacters = ref();

const characters = ref([
  { "name": "Aria", "race": "Half-Elf", "className": "Rogue", "level": 5, "createdAt": "2021-09-01" },
  { "name": "Marcus", "race": "Human", "className": "Paladin", "level": 1, "createdAt": "2021-09-15" },
  { "name": "Tessa", "race": "Dwarf", "className": "Bard", "level": 8, "createdAt": "2021-09-22" },
  { "name": "Erik", "race": "Dwarf", "className": "Barbarian", "level": 3, "createdAt": "2021-10-01" },
  { "name": "Lila", "race": "Tiefling", "className": "Warlock", "level": 7, "createdAt": "2021-10-07" },
  { "name": "Niamh", "race": "Elf", "className": "Ranger", "level": 2, "createdAt": "2021-10-15" },
  { "name": "Tristan", "race": "Half-Orc", "className": "Fighter", "level": 6, "createdAt": "2021-10-23" },
  { "name": "Ava", "race": "Halfling", "className": "Cleric", "level": 4, "createdAt": "2021-11-01" }
]);
const character = ref({});

const openNew = () => {
  character.value = {};
  submitted.value = false;
  characterDialog.value = true;
};

const hideDialog = () => {
  characterDialog.value = false;
  submitted.value = false;
};

const saveCharacter = () => {
  submitted.value = true;

  if (character?.value.name?.trim()) {
      if (character.value.createdAt) {
          characters.value = characters.value.map(c => c.name === character.value.prevName ? { ...c, ...character.value } : c);
          toast.add({severity:'success', summary: 'Successful', detail: 'Character Updated', life: 3000});
      }
      else {
          characters.value.push({ ...character.value, createdAt: new Date().toISOString().slice(0, 10) });
          toast.add({severity:'success', summary: 'Successful', detail: 'Character Created', life: 3000});
      }

      characterDialog.value = false;
      character.value = {};
  }
};

const editCharacter = char => {
  character.value = {...char, prevName: char.name};
  characterDialog.value = true;
};

const confirmDeleteCharacter = char => {
  character.value = char;
  deleteCharacterDialog.value = true;
};

const deleteCharacter = () => {
  characters.value = characters.value.filter(val => val.name !== character.value.name);
  deleteCharacterDialog.value = false;
  character.value = {};
  toast.add({severity:'success', summary: 'Successful', detail: 'Character Deleted', life: 3000});
};

const confirmDeleteSelected = () => {
  deleteCharactersDialog.value = true;
};

const deleteSelectedCharacters = () => {
  characters.value = characters.value.filter(char => !selectedCharacters.value.find(selectedChar => selectedChar.name === char.name));
  deleteCharactersDialog.value = false;
  selectedCharacters.value = null;
  toast.add({severity:'success', summary: 'Successful', detail: 'Characters Deleted', life: 3000});
};
</script>
