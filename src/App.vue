<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";

const singleFile = ref<File | null>(null);
const viewImg = ref<string | null>(null);
const singleName = ref<string | null>(null);
const multipleFiles = ref<File[]>([]);

const singleUpload = async () => {
  if (singleFile.value) {
    try {
      const response = await uploadSingle(singleFile.value);
      console.log("single", singleFile);
      const url = response.data.objectName as string;
      viewImg.value = getFileImageUrl(singleFile.value);
    } catch (error) {
      console.log(error);
    }
  }
};

async function uploadSingle(file: File) {
  const data = new FormData();
  data.append("tenantId", "test");
  data.append("module", "test");
  data.append("fileName", file.name);
  data.append("file", file);

  return await axios.post(
    "http://192.168.100.241:9999/api/file/upload/public",
    data
  );
}

function singleChange(e: any) {
  if (e.target.files && e.target.files.length > 0) {
    singleFile.value = e.target.files[0];
    singleName.value = e.target.files[0].name;
    viewImg.value = getFileImageUrl(e.target.files[0]);
  }
}

function getFileImageUrl(file: File): string {
  if (file.type.includes("jpeg") || file.type.includes("png")) {
    return URL.createObjectURL(file);
  } else if (file.type.includes("pdf")) {
    return "https://play-lh.googleusercontent.com/9XKD5S7rwQ6FiPXSyp9SzLXfIue88ntf9sJ9K250IuHTL7pmn2-ZB0sngAX4A2Bw4w";
  } else if (file.type.includes("zip")) {
    return "https://d1nhio0ox7pgb.cloudfront.net/_img/g_collection_png/standard/512x512/folder_zip.png";
  } else if (file.type.includes("sql")) {
    return "https://www.shareicon.net/data/2015/09/07/97430_document_512x512.png";
  } else if (file.type.includes("html")) {
    return "https://cdn4.iconfinder.com/data/icons/smashicons-file-types-flat/56/22_-_HTML_File_Flat-512.png";
  } else {
    return "https://www.iconpacks.net/icons/2/free-file-icon-1453-thumb.png";
  }
}

const addFile = (index: number, event: any) => {
  if (event.target.files && event.target.files.length > 0) {
    multipleFiles.value[index] = event.target.files[0];
  }
};

const multipleUpload = async () => {
  try {
    await Promise.all(multipleFiles.value.map(uploadFile));
    console.log("All files uploaded successfully");
  } catch (error) {
    console.error("Error uploading files:", error);
  }
};

async function uploadFile(file: File) {
  console.log("Uploading file:", file);
  const data = new FormData();
  data.append("tenantId", "test");
  data.append("module", "test");
  data.append("fileName", file.name);
  data.append("file", file);

  return await axios.post(
    "http://192.168.100.241:9999/api/file/upload/public",
    data
  );
}
</script>

<template>
  <div>
    <p class="font-bold text-purple-600">Single File Upload</p>
    <div
      class="flex m-1 flex-col justify-between items-center w-64 h-64 border rounded-xl border-purple-800"
    >
      <label>
        <img
          class="w-56 m-2 h-56"
          :src="
            viewImg ||
            'https://www.shutterstock.com/image-vector/download-icon-vector-illustration-on-600nw-1724759407.jpg'
          "
          alt="file or image not found"
        />
        <input class="hidden" type="file" @change="singleChange" />
      </label>
    </div>
    <button
      class="w-64 m-1 bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded-lg"
      @click="singleUpload"
      type="button"
    >
      Upload
    </button>
  </div>

  <div>
    <p class="font-bold text-purple-600">Multiple File Upload</p>
    <div
      class="flex m-1 flex-col justify-between items-center w-64 h-64 border rounded-xl border-purple-800"
    >
      <label v-for="(file, index) in multipleFiles" :key="index">
        <img
          class="w-56 m-2 h-56"
          :src="
            getFileImageUrl(file) ||
            'https://www.shutterstock.com/image-vector/download-icon-vector-illustration-on-600nw-1724759407.jpg'
          "
          alt="file or image not found"
        />
        <input class="hidden" type="file" @change="addFile(index, $event)" />
      </label>
    </div>
    <button
      class="w-64 m-1 bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded-lg"
      @click="multipleUpload"
      type="button"
    >
      Upload
    </button>
  </div>
</template>

<style>
.btn {
  @apply font-bold py-2 px-4 rounded;
}

.btn-blue {
  @apply bg-blue-500 text-white;
}

.btn-blue:hover {
  @apply bg-blue-700;
}

.buttons {
  width: 200px;
  height: 200px;
  background: url("https://www.shutterstock.com/image-vector/download-icon-vector-illustration-on-600nw-1724759407.jpg");
}
</style>
