<template>
  <div>
    <h3>Files</h3>
    <p v-for="(obj, index) in files" :key="(index + 1) * Math.random()">
      {{ obj.name }}
    </p>

    <button @click="getAllFiles">GETALL FILES</button>
    <ul v-for="(file, index) in results" :key="(index + 1) * Math.random()">
      <li @click="fetchFile({ filename: file.filename, id: file._id })">
        {{ file.filename }}
        {{ file._id }}
      </li>
    </ul>
    <h2>Multiple files</h2>
    <form method="post" enctype="multipart/form-data">
      <input type="file" multiple @change="handleUpload" name="avatar" />
      <input @click="postFile" type="submit" value="Submit" />
    </form>

    <p v-for="(obj, index) in uploadFiles" :key="(index + 1) * Math.random()">
      -------- {{ obj.file }} --------
      {{ obj }}
      {{ uploadFiles }}
    </p>
  </div>
</template>

<script>
const URI = "http://localhost:3000";
// const URI = "http://localhost:4000";
export default {
  data() {
    return {
      files: [],
      inputFile: {},
      uploadFiles: {},
      results: []
    };
  },
  methods: {
    handleUpload(e) {
      console.log(this.files);
      Object.values(e.target.files).map(d => this.files.push(d));
    },

    uploadFile(file) {
      const formData = new FormData();
      formData.append("uploadedFile", file, file.name);
      fetch(`${URI}/postFile`, {
        method: "POST",
        body: formData
      })
        .then(function(response) {
          console.log("RESPONSE", response);
          if (response.ok) {
            console.log("Click was recorded");
            return;
          }
          throw new Error("Request failed.");
        })
        .catch(function(error) {
          console.log(error);
        });
    },

    postFile(e) {
      e.preventDefault();
      this.files.map(file => {
        this.uploadFile(file);
      });
      this.files = [];
    },

    async fetchFile({ filename, id }) {
      const filePath = `${URI}/getFile?id=${id}`;
      const respData = await fetch(filePath, {
        method: "GET"
      });
      const blob = await respData.blob();
      console.log(blob);
      var url = window.URL.createObjectURL(blob);
      var a = document.createElement("a");
      a.href = url;
      a.download = `${filename}`;
      document.body.appendChild(a); // we need to append the element to the dom -> otherwise it will not work in firefox
      a.click();
      a.remove(); //afterwards we remove the element again
    },

    async getAllFiles() {
      const resp = await fetch(`${URI}/listFiles`, {
        method: "GET",
        headers: {
          "Content-Type": "application/json"
        }
      });
      const data = await resp.json();
      this.results = data;
    }
  }
};
</script>
