<script setup lang="ts">
  import axios from 'axios';
  import { useToast } from "primevue/usetoast";

  const baseUrl = import.meta.env.VITE_BASEURL;
  const toast = useToast();
  const totalSize = ref(0);
  const totalSizePercent = ref(0);
  const files = ref<File[]>([]);
  const generateObjectURL = (file: File): string => {
    return URL.createObjectURL(file);
  };

  const onRemoveTemplatingFile = (file: any, removeFileCallback: any, index: any) => {
    removeFileCallback(index);
    totalSize.value -= file.size; // Adjust calculation
    totalSizePercent.value = (totalSize.value / 500000000) * 100; // 500MB in bytes
  };

  const onSelectedFiles = (event: any) => {
    files.value = event.files;
    if (files.value) {
      files.value.forEach((file: any) => {
        totalSize.value += file.size;
      });
      totalSizePercent.value = (totalSize.value / 500000000) * 100; // 500MB in bytes
    }
  };

  const uploadEvent = (callback: any) => {
    totalSizePercent.value = totalSize.value / 5000000; // Adjusted for 500MB
    callback();
  };

  const onTemplatedUpload = () => {
    toast.add({ severity: "success", summary: "Success", detail: "File Uploaded", life: 3000 });
  };

  const formatSize = (bytes: any) => {
    const k = 1024;
    const dm = 2; // Adjusted decimal places
    const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];

    if (bytes === 0) return `0 ${sizes[0]}`;

    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return `${parseFloat((bytes / Math.pow(k, i)).toFixed(dm))} ${sizes[i]}`;
  };


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
      toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to download files.', life: 3000 });
    } 
  };

  const clearFiles = async () => {
    try {
      await axios.post(`${baseUrl}/clear`);
      toast.add({ severity: 'info', summary: 'Success', detail: 'All files have been cleared from server.', life: 3000 });
    } catch (error) {
      console.error('Failed to clear files:', error);
      toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to clear files.', life: 3000 });
    }
  };

  const isDarkMode = ref(false);
  const iconClass = computed(() => isDarkMode.value ? 'pi pi-sun' : 'pi pi-moon');

  function toggleDarkMode() {
      const element = document.querySelector('html');
      if (element) {
        element.classList.toggle('my-app-dark');
        isDarkMode.value = !isDarkMode.value;
      }
  }

  const getObjectURL = (file: File) => {
    return URL.createObjectURL(file);
  };

</script>

<template>
  <div class="bg-slate-700 w-full h-full flex flex-column justify-content-start align-items-center container">
    <Toast />
    <div class="w-full flex justify-content-start"><Button class="mx-5 my-3" @click="toggleDarkMode()"><i :class="iconClass" style="font-size: 1.5rem"></i></Button></div>
    <div :class="{'app_content_dark': isDarkMode}" class="w-10 h-10 app_content_light">
        <div class="flex justify-content-center align-items-center"><h1 class="mr-2">Pi-Share</h1><i class="pi pi-send" style="font-size: 2.7rem;"></i></div>
        <FileUpload class="mx-10" name="files" :url="`${baseUrl}/upload`" @upload="onTemplatedUpload()" :multiple="true" :maxFileSize="524288000" @select="onSelectedFiles"> <!-- Updated maxFileSize to 500MB -->
          <template #header="{ chooseCallback, uploadCallback, clearCallback, files }">
              <div class="flex flex-wrap justify-between items-center flex-1 gap-4">
                  <div class="flex gap-2">
                      <Button @click="chooseCallback()" icon="pi pi-images" rounded outlined severity="secondary"></Button>
                      <Button @click="uploadEvent(uploadCallback)" icon="pi pi-cloud-upload" rounded outlined severity="success" :disabled="!files || files.length === 0"></Button>
                      <Button @click="clearCallback()" icon="pi pi-times" rounded outlined severity="danger" :disabled="!files || files.length === 0"></Button>
                  </div>
                  <ProgressBar :value="totalSizePercent" :showValue="false" class="md:w-20rem h-1 w-full md:ml-auto">
                      <span class="whitespace-nowrap">{{ totalSize }}B / 500Mb</span> <!-- Updated display to 500MB -->
                  </ProgressBar>
              </div>
          </template>
          <template #content="{ files, uploadedFiles, removeUploadedFileCallback, removeFileCallback }">
              <div class="flex flex-col gap-8 pt-4">
                  <div v-if="files.length > 0">
                      <h5>Pending</h5>
                      <div class="flex flex-wrap gap-4">
                        <div v-for="(file, index) of files" :key="file.name + file.type + file.size" class="p-8 rounded-border flex flex-col border border-surface items-center gap-4">
                          <div>
                            <img role="presentation" :alt="file.name" :src="getObjectURL(file)" width="100" height="50" />
                          </div>
                          <span class="font-semibold text-ellipsis max-w-60 whitespace-nowrap overflow-hidden">{{ file.name }}</span>
                          <div>{{ formatSize(file.size) }}</div>
                          <Badge value="Pending" severity="warn" />
                          <Button icon="pi pi-times" @click="onRemoveTemplatingFile(file, removeFileCallback, index)" outlined rounded severity="danger" />
                        </div>
                      </div>
                  </div>

                  <div v-if="uploadedFiles.length > 0">
                      <h5>Completed</h5>
                      <div class="flex flex-wrap gap-4">
                          <div v-for="(file, index) of uploadedFiles" :key="file.name + file.type + file.size" class="p-8 rounded-border flex flex-col border border-surface items-center gap-4">
                              <div>
                                  <img role="presentation" :alt="file.name" :src="generateObjectURL(file)" width="100" height="50" />
                              </div>
                              <span class="font-semibold text-ellipsis max-w-60 whitespace-nowrap overflow-hidden">{{ file.name }}</span>
                              <div>{{ formatSize(file.size) }}</div>
                              <Badge value="Completed" class="mt-4" severity="success" />
                              <Button icon="pi pi-times" @click="removeUploadedFileCallback(index)" outlined rounded severity="danger" />
                          </div>
                      </div>
                  </div>
              </div>
          </template>
          <template #empty>
              <div class="flex items-center justify-center flex-col drop-box">
                  <i class="pi pi-cloud-upload !border-2 !rounded-full !p-8 !text-4xl !text-muted-color" />
                  <p class="mt-6 mb-0">Drag and drop files to here to upload.</p>
              </div>
          </template>
        </FileUpload>
        <div class="flex justify-content-around">
          <Button @click="downloadFiles" severity="info" class="m-3" size="large"><i class="pi pi-download"></i>Download</Button>
          <Button @click="clearFiles" severity="danger" class="m-3" size="large"><i class="pi pi-trash"></i>Clear</Button>
        </div>
    </div>
  </div>
</template>

<style scoped>
  h1 {
    font-weight: 300;
    font-size: 3.5rem;
  }
  .app_content_light {
    background-color: var(--p-slate-200);
    border-radius: .5rem;
    box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.2);
    height: 85vh;
    padding: 3%;
    overflow: auto;
  }
  .app_content_dark {
    background-color: var(--p-slate-800);
    border-radius: .5rem;
    box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.2);
    height: 85vh;
    padding: 3%;
    overflow: auto;
  }
  .container {
    margin: 0%;
    padding: 0%;
    width: 100%;
    height: 100vh;
    background-color: var(--p-slate-600);
  }
  .drop-box{
    height: 28vh;
  }
</style>
