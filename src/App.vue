<template>
  <div id="app">
    <div class="addDir">
      <input v-model="dirName" type="text" />
      <button @click="addDir">Add dir</button>
    </div>
    <br />
    <div class="uploadFile">
      <input type="file" @change="onFileChange($event)" />
      <button>Upload file</button>
    </div>

    <ul>
      <li v-for="(c, index) in contents" :key="index">
        <p v-if="c.is_dir">+</p>
        <p v-else></p>

        <p @dblclick="goToFolder(c)" class="fileName">{{ c.file }}</p>
        <button>x</button>
        <button @click="showEditField(index)">/</button>
        <div class="edit" v-if="edit && currentEditableIndex == index">
          <input v-model="fileName" type="text" />
          <button @click="editFileName(c)" style="width: 50px">edit</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "app",
  data() {
    return {
      contents: [],
      edit: false,
      fileName: "",
      currentDir: "",
      currentEditableIndex: -1,
      dirName: ""
    };
  },
  components: {},

  created() {
    axios.get("http://localhost:3000").then(res => {
      this.contents = res.data.contents;
      this.currentDir = res.data.currentDir;
    });
  },

  methods: {
    editFileName: function(file) {
      console.log(file + ", new-name: " + this.fileName);
      axios
        .post("http://localhost:3000/changeName", {
          oldPath: file.path,
          newPath: file.dir + "/" + this.fileName
        })
        .then(res => {
          this.contents = res.data.contents;
          this.currentDir = res.data.currentDir;
        });
      this.edit = false;
    },

    showEditField: function(index) {
      if (index == this.currentEditableIndex) {
        this.edit = !this.edit;
      } else {
        this.edit = true;
      }

      this.currentEditableIndex = index;
    },

    onFileChange: function(e) {
      console.log(e.target.files[0]);
    },

    addDir: function() {
      console.log(this.currentDir);
      console.log(this.dirName);
      axios
        .post("http://localhost:3000/addDir", {
          currentDir: this.currentDir,
          dirName: this.dirName
        })
        .then(res => {
          this.contents = res.data.contents;
          this.currentDir = res.data.currentDir;
        });
    },

    goToFolder(file) {
      if (file.is_dir) {
        console.log(file);
      } else {
        //window.open(`${file.dir}/${file.file}`, "_blank");

        let pathWithoutUser = file.dir
          .split("/")
          .slice(3)
          .join("/");

        var server = "127.0.0.1:8887";
        window.open(
          `http://${server}/${pathWithoutUser}/${file.file}`,
          "_blank"
        );
      }
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

#app li {
  list-style-type: none;
  display: grid;
  grid-template-columns: 10px minmax(100px, 200px) 30px 30px 300px;
  grid-gap: 10px;
  border: 2px solid pink;
  padding: 0px 10px;
  align-items: center;
}

button {
  background: #333;
  color: #fff;
  border: none;
  height: 30px;
  width: auto;
  font-size: 1.2rem;
}

button:hover {
  cursor: pointer;
}

.edit {
  width: 300px;
  display: grid;
  grid-template-columns: 1fr 1fr;
}

.addDir {
  width: 500px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  margin-left: 40px;
}

.uploadFile {
  display: grid;
  grid-template-columns: minmax(100px, 300px) minmax(100px, 200px);
  margin-left: 40px;
}

.fileName {
  border: 2px solid pink;
}

.fileName:hover {
  cursor: pointer;
}
</style>
