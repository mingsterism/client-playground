<template>
  <div>
    <form method="post" enctype="multipart/form-data">
      <input type="file" name="avatar" @change="handleUpload" />
      <input @click="postingFile" type="submit" value="Submit" />
    </form>

    <button @click="getAllFiles">GETALL FILES</button>
    <ul v-for="(file, index) in files" :key="(index + 1) * Math.random()">
      <li @click="fetchFile({ filename: file.filename, id: file._id })">
        {{ file.filename }}
        {{ file._id }}
      </li>
    </ul>
    <h2>Multiple files</h2>
    <form method="post" enctype="multipart/form-data">
      <input type="file" multiple @change="handleUpload" name="avatar" />
      <input @click="postMultipleFiles" type="submit" value="Submit" />
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
      uploadFiles: {}
    };
  },
  methods: {
    handleInput(e) {
      console.log("handle Input");
      console.log(e.target.files);
    },
    handleChange(e) {
      console.log("handleChange");
      console.log(e.target.files);
      this.uploadFiles = e.target.files;
    },
    uploadImage(e) {
      this.inputFile = e.target.files[0];
      console.log(e.target.files[0]);
    },
    postMultipleFiles(e) {
      e.preventDefault();
      console.log("----------");
      console.log(e);
      console.log(e.target.files);
      console.log("FILE LIST: ", this.uploadFiles);
      Object.values(this.uploadFiles).map(file => this.postFile(file));
    },
    handleUpload(e) {
      e.preventDefault();
      console.log(typeof e.target.files);
      if (e.target.files.length > 1) {
        console.log("MULTI");
        console.log(e.target.files.length);
        console.log(e.target.files);
        this.files = e.target.files;
      } else {
        console.log("Single");
        console.log(e.target.files);
        this.files = e.target.files;
      }

      // this.uploadFiles(e.target.files[0]);
    },
    uploadFile(file) {
      console.log("Uploading file");
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

    postingFile(e) {
      e.preventDefault();
      if (this.files.length > 1) {
        console.log("MULTI");
        console.log(e.target.files.length);
        console.log(e.target.files);
        Object.values(this.files).map(file => this.uploadFile(file));
      } else {
        console.log("Single");
        console.log(this.files);
        this.uploadFile(this.files);
      }
    },
    postFile(e) {
      e.preventDefault();
      const formData = new FormData();
      formData.append("uploadedFile", this.inputFile, this.inputFile.name);
      fetch(`${URI}/postFile`, {
        method: "POST",
        body: formData
      })
        .then(function(response) {
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
      this.files = data;
    },

    async test() {
      const resp = await fetch(`${URI}/test`, {
        method: "GET",
        headers: {
          "Content-Type": "application/json"
        }
      });
      const data = await resp.json();
      console.log(data);
    }
  }
};
</script>
