<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";

interface MultiResList {
  id: string;
  file: File;
  size: number;
  img: string;
  name: string;
  status: number;
  objectName: string;
}

const multipleFiles = ref<MultiResList[]>([]);
const imgUrl = ref<string>("");
const multiLoader = ref(false);
const isDisabled = ref(false);
const limit = ref<number>(10000000);

function multiChange(e: Event) {
  const files = (e.target as HTMLInputElement).files;

  if (!files) return;

  for (let i = 0; i < files.length; i++) {
    const item = files[i];
    if (item.type.includes("jpeg") || item.type.includes("png")) {
      imgUrl.value = URL.createObjectURL(item);
    } else if (item.type.includes("pdf")) {
      imgUrl.value =
        "https://play-lh.googleusercontent.com/9XKD5S7rwQ6FiPXSyp9SzLXfIue88ntf9sJ9K250IuHTL7pmn2-ZB0sngAX4A2Bw4w";
    } else if (item.type.includes("zip")) {
      imgUrl.value =
        "https://d1nhio0ox7pgb.cloudfront.net/_img/g_collection_png/standard/512x512/folder_zip.png";
    } else if (item.type.includes("sql")) {
      imgUrl.value =
        "https://www.shareicon.net/data/2015/09/07/97430_document_512x512.png";
    } else if (item.type.includes("html")) {
      imgUrl.value =
        "https://cdn4.iconfinder.com/data/icons/smashicons-file-types-flat/56/22_-_HTML_File_Flat-512.png";
    } else if (item.type.includes("exe")) {
      imgUrl.value =
        "https://t3.ftcdn.net/jpg/02/45/28/26/360_F_245282659_xPkY916cZ50k7bx9objepCrbtmColpCY.jpg";
    } else {
      imgUrl.value =
        "https://www.iconpacks.net/icons/2/free-file-icon-1453-thumb.png";
    }

    multipleFiles.value.push({
      id: new Date().getTime() + i.toString(),
      file: item,
      size: item.size,
      img: imgUrl.value,
      name: item.name,
      status: 0,
      objectName: "",
    });
  }
}

const multipleUpload = async () => {
  if (!multipleFiles) return;
  multiLoader.value = true;
  for (let i = 0; i < multipleFiles.value.length; i++) {
    try {
      if (multipleFiles.value[i].status !== 1) {
        if (multipleFiles.value[i].size < limit.value) {
          const response = await uploadSingle(multipleFiles.value[i].file);
          multipleFiles.value[i].objectName = response.data.objectName;

          if (response.status == 200) {
            multipleFiles.value[i].status = 1;
          }
        } else {
          multipleFiles.value[i].status = 3;
        }
      }
    } catch (error) {
      multipleFiles.value[i].status = 2;
      multiLoader.value = false;
    }
  }
  multiLoader.value = false;
};

async function uploadSingle(file: File) {
  const data = new FormData();
  data.append("tenantId", "test");
  data.append("module", "test");
  data.append("fileName", "test" + file.name);
  data.append("file", file);

  const res = await axios.post(
    "http://192.168.100.241:9999/api/file/upload/public",
    data
  );

  return res;
}

const remove = async (id: string) => {
  multipleFiles.value = multipleFiles.value.filter((e) => e.id !== id);
};
</script>

<template>
  <div class="flex flex-col gap-3 w-[1300px] h-[100%] p-2">
    <b class="flex items-center text-purple-900">Multiple Upload</b>
    <div class="flex flex-wrap gap-2">
      <label class="buttons border relative flex items-center">
        <b class="p-3 text-red-500 bottom-0 absolute">Max file size - 10MB</b>
        <input class="hidden" type="file" multiple @change="multiChange" />
        <button
          :disabled="isDisabled"
          @click="multipleUpload"
          class="bg-blue-500 active:bg-green-600 text-white absolute bottom-[-55px] left-0 w-[200px] border-none"
        >
          {{ multiLoader ? "Saving..." : "Save" }}
        </button>
      </label>
      <div class="flex flex-wrap" v-for="item in multipleFiles" :key="item.id">
        <div
          class="relative h-auto p-8 w-[220px] border flex flex-col justify-center items-center"
        >
          <img :src="item.img" class="w-[200px] h-auto" />
          <img
            v-if="item.status == 1"
            class="absolute top-0 left-1 w-[30px]"
            src="https://www.shutterstock.com/image-vector/green-check-mark-icon-tick-600nw-522874111.jpg"
          />
          <img
            v-else-if="item.status == 2"
            class="absolute top-1 left-1 w-[30px]"
            src="https://t3.ftcdn.net/jpg/04/83/27/52/360_F_483275230_S6o73rQ6Brygo92tDwoR4TLwdGxwzPMw.jpg"
          />
          <img
            v-else-if="item.status == 3"
            class="absolute top-1 right-1 w-[30px]"
            src="https://cdn.iconscout.com/icon/premium/png-256-thumb/max-size-7251675-5965202.png?f=webp"
          />
          <b class="w-[150px]">{{ item.name }}</b>
          <button
            @click="remove(item.id)"
            class="bg-red-600 bottom-0 absolute text-white w-[200px] text-[12px] rounded border-none"
          >
            Remove
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
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
  background-image: url("data:image/png;base64,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");
  background-position: center;
  background-size: cover;
  line-height: normal;
}
</style>
