<template>
  <form
    class="dropbox"
    @dragenter.prevent.stop
    @dragover.prevent.stop
    @drop="handleDrop"
  >
    <p class="dropbox__text">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        width="64"
        height="64"
      >
        <path fill="none" d="M0 0h24v24H0z" />
        <path
          d="M21 15v3h3v2h-3v3h-2v-3h-3v-2h3v-3h2zm.008-12c.548 0 .992.445.992.993V13h-2V5H4v13.999L14 9l3 3v2.829l-3-3L6.827 19H14v2H2.992A.993.993 0 0 1 2 20.007V3.993A1 1 0 0 1 2.992 3h18.016zM8 7a2 2 0 1 1 0 4 2 2 0 0 1 0-4z"
          fill="rgba(204,204,204,1)"
        />
      </svg>
      Upload multiple files by clicking the link below or by dragging and
      dropping images onto the dashed region
    </p>
    <input
      ref="inputFile"
      type="file"
      multiple
      accept="image/*"
      @change="handleFileInput"
      style="display:none"
    />
    <button type="button" @click="fileSelect">
      Select some files
    </button>
  </form>
  <div v-if="progress" className="progress">
    <div className="progress__inner" :style="{ width: `${progress}%` }"></div>
  </div>
</template>

<script>
import { ref } from "vue";
import axios from "axios";
export default {
  name: "Dropbox",
  setup() {
    const progress = ref(0);
    const inputFile = ref(null);

    const fileSelect = () => {
      inputFile.value.click();
    };

    const uploadFile = async file => {
      const formData = new FormData();

      formData.append("file", file);
      formData.append(
        "upload_preset",
        process.env.VUE_APP_CLOUDINARY_UPLOAD_PRESET
      );

      // Send to cloudianry
      const { data } = await axios.post(
        process.env.VUE_APP_CLOUDINARY_URL,
        formData,
        {
          headers: {
            "Content-Type": "multipart/form-data"
          },
          onUploadProgress(e) {
            progress.value = Math.round((e.loaded * 100.0) / e.total);
          }
        }
      );

      console.log(data);
      alert("Success upoload");
      progress.value = 0;
    };

    const handleFiles = files => {
      for (let i = 0; i < files.length; i++) uploadFile(files[i]);
    };

    const handleFileInput = ({ target }) => handleFiles(target.files);

    const handleDrop = e => {
      e.stopPropagation();
      e.preventDefault();

      const dt = e.dataTransfer;
      const files = dt.files;

      handleFiles(files);
    };
    return { progress, inputFile, fileSelect, handleFileInput, handleDrop };
  }
};
</script>

<style lang="scss" scoped>
.dropbox {
  border: 4px dashed #ccc;
  padding: 1rem;
  &__text {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }
}
.progress {
  width: 100%;
  height: 4px;
  background-color: #e2dfdf;
  margin: 1rem auto;
  &__inner {
    background-color: #ccc;
    height: 100%;
  }
}
</style>
