<template>
  <div>
    <form method="post" enctype="multipart/form-data">
      <input type="file" name="avatar" @change="uploadImage" />
      <input @click="postFile" type="submit" value="Submit" />
    </form>
    <button @click="getAllFiles">GETALL FILES</button>
    <ul v-for="(file, index) in files" :key="(index + 1) * Math.random()">
      <li @click="fetchFile({ filename: file.filename, id: file._id })">
        {{ file.filename }}
        {{ file._id }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      files: [],
      inputFile: {}
    };
  },
  methods: {
    uploadImage(e) {
      this.inputFile = e.target.files[0];
    },
    postFile(e) {
      e.preventDefault();
      const formData = new FormData();
      formData.append("uploadedFile", this.inputFile, this.inputFile.name);
      fetch("http://localhost:3000/postFile", {
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
      const filePath = `http://localhost:3000/getFile?id=${id}`;
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
      const resp = await fetch("http://localhost:3000/listFiles", {
        method: "GET",
        headers: {
          "Content-Type": "application/json"
        }
      });
      const data = await resp.json();
      this.files = data;
    },

    async test() {
      const resp = await fetch("http://localhost:4000/test", {
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
