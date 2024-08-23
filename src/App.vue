<script setup lang="ts">
  import axios from 'axios';
  import { useToast } from "primevue/usetoast";

  const baseUrl = import.meta.env.VITE_BASEURL;
  const toast = useToast();
  const fileupload = ref();


  const downloadFiles = async () => {
    try {
      const response = await axios.get(`${baseUrl}/download`, {
        responseType: 'blob',
      });
      const url = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', 'all_files.zip');
      document.body.appendChild(link);
      link.click();
      link.remove();
      toast.add({ severity: 'success', summary: 'Success', detail: 'Files downloaded.', life: 3000 });
    } catch (error) {
      console.error('File download failed:', error);
      toast.add({ severity: 'danger', summary: 'Error', detail: 'Failed to download files.', life: 3000 });
    } 
  };

  const clearFiles = async () => {
    try {
      await axios.post(`${baseUrl}/clear`);
      toast.add({ severity: 'info', summary: 'Success', detail: 'All files have been cleared from server.', life: 3000 });
    } catch (error) {
      console.error('Failed to clear files:', error);
      toast.add({ severity: 'danger', summary: 'Error', detail: 'Failed to clear files.', life: 3000 });
    }
  };

  const upload = () => {
      fileupload.value.upload();
  };

  const onUpload = () => {
      toast.add({ severity: 'success', summary: 'Success', detail: 'Files Uploaded', life: 3000 });
  };

  const isDarkMode = ref(false);
  const iconClass = computed(() => isDarkMode.value ? 'pi pi-sun' : 'pi pi-moon');

  function toggleDarkMode() {
      const element = document.querySelector('html');
      element.classList.toggle('my-app-dark');
      isDarkMode.valie = !isDarkMode;
  }
</script>

<template>
  <div class="w-full h-full flex flex-column justify-content-start align-items-center container">
    <Toast />
    <div class="w-full flex justify-content-end"><Button class="mx-5 my-3" @click="toggleDarkMode()"><i :class="iconClass" style="font-size: 1.5rem"></i></Button></div>
    <Card class="w-10 h-10">
      <template #title>
        <div class="flex justify-content-center"><h1>File Manager</h1></div>
      </template>
      <template #content>
        <FileUpload ref="fileupload" mode="basic" name="files" :url="`${baseUrl}/upload`" :maxFileSize="1000000" @upload="onUpload" :multiple="true" />
        <Button label="Upload" @click="upload" severity="secondary" />
        <Button @click="downloadFiles" severity="success">Download All Files</Button>
        <Button @click="clearFiles" severity="danger">Clear All Files</Button>
      </template>
    </Card>
  </div>
</template>

<style scoped>
  h1 {
    font-weight: 300;
    font-size: 3.5rem;
  }
  .container {
    background-color: aquamarine;
  }
</style>
