<template>
  <div class="container mb-2" style="background-image: linear-gradient(to right top, #e2e6ed, #d4e9f2, #c7edeb, #c8eed8, #ddeac0);">
    <div class="form bg-text">
      <h4>Upload a new photo</h4>
      <form @submit.prevent="onUploadPhoto" method="post">
        <div class="form-group">
          <label class="control-label">Title</label>
          <input
            type="text"
            class="form-control"
            v-model="photoName"
            required
          />
        </div>
        <div class="form-group">
          <label class="control-label">Select Picture</label>
          <input
            type="file"
            class="form-control"
            @change="onFileSeleted"
            accept="image/*"
            required
          />
        </div>
        <div class="multiselect-div">
          <label class="typo__label" for="ajax">Select 2 Category</label>
          <multiselect
            v-model="value"
            id="ajax"
            label="categoryName"
            track-by="categoryId"
            placeholder="Type to search"
            open-direction="bottom"
            :options="options"
            :multiple="true"
            :searchable="true"
            :internal-search="true"
            :clear-on-select="false"
            :close-on-select="false"
            :options-limit="100"
            :limit="2"
            :max-height="150"
            :hide-selected="true"
            :max="2"
          >
          </multiselect>
        </div>
        <div class="multiselect-div">
          <label class="typo__label">License Type</label>
          <multiselect
            v-model="type"
            deselect-label="Can't remove this value"
            open-direction="bottom"
            track-by="typeId"
            label="typeName"
            placeholder="Select one"
            :options="typeList"
            :searchable="false"
            :allow-empty="false"
          >
          </multiselect>
        </div>
        <div class="form-group">
          <label class="control-label">Description</label>
          <input
            type="text"
            class="form-control"
            v-model="description"
            required
          />
        </div>
        <div class="form-group">
          <label class="control-label">Price</label>
          <input type="text" class="form-control" v-model="price" required />
        </div>
        <div class="buttonHolder">
          <button class="btn btn-primary submit-button" type="submit">
            <div class="lds-ring" v-if="loading">
              <div></div>
              <div></div>
              <div></div>
              <div></div>
            </div>
            Upload
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";
import Spinner from "./Spinner.vue";
import Modal from "@/components/Modal.vue";
export default {
  components: {
    Multiselect,
    Spinner,
    Modal,
  },
  data() {
    return {
      modals: {
        modal1: false,
        modal2: false,
        modal3: false,
      },
      isImage: false,
      upload: false,
      loading: false,
      photoName: "",
      file: null,
      price: "",
      description: "",
      value: [],
      options: [],
      type: null,
      typeList: [],
      user: JSON.parse(localStorage.getItem("user")),
    };
  },
  mounted: function() {
    axios
      .get("https://imago.azurewebsites.net/api/v1/Category")
      .then((response) => {
        this.options = response.data;
        this.clone_options = response.data;
      });
    axios
      .get("https://imago.azurewebsites.net/api/v1/Type/GetAllType")
      .then((response) => {
        this.typeList = response.data;
      });
  },
  methods: {
    onFileSeleted(event) {
      this.file = event.target.files[0];
      if(!this.file.type.match('image.*')) {
        alert('Invalid image');
        this.isImage = false;
      } 
      if(this.file.type.match('image.*')) {
        this.isImage = true;
      }
      console.log(this.isImage);
    },
    onUploadPhoto() {
      this.upload = true;
      this.loading = true;
      if (this.value.length === 0) {
        alert("Please select category");
        return;
      }
      if (this.isImage === false) {
        alert("Please select an image");
        return;
      }
      if (isNaN(this.price)) {
        alert("Please enter only the number to the price");
        return;
      }
      const fd = new FormData();
      fd.append("photoName", this.photoName);
      fd.append("file", this.file, this.file.name);
      fd.append("price", this.price);
      fd.append("typeId", this.type.typeId);
      fd.append("userId", this.user.userId);
      fd.append("listCategory", this.value[0].categoryId);
      if (this.value.length === 2) {
        fd.append("listCategory", this.value[1].categoryId);
      }

      fd.append("description", this.description);
      let config = {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      };

      axios
        .post(
          "https://imago.azurewebsites.net/api/v1/Photo/CreatePhoto",
          fd,
          config
        )
        .then((respone) => {
          if (respone.status == 201) {
            this.loading = false;
            this.$alert(
              "Upload Successfully",
              "Success",
              "success"
            ).then(() => console.log("Closed"));
           
          } else {
            alert("Upload error");
          }
          console.log(respone.status);
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>

<style lang="scss" scoped>

.container {
  border: 1px solid;
  border-radius: 12px;
  max-width: 60%;
}
div.form {
  display: block;
  text-align: center;
}
form {
  display: block;
  margin-left: auto;
  margin-right: auto;
  text-align: left;
  max-width: 300px;
}
.buttonHolder {
  text-align: center;
}
h4 {
  padding-top: 5%;
}
.multiselect-div {
  margin-bottom: 28px;
}

.submit-button {
  width: 170px;
  margin-bottom: 5px;
}

.lds-ring {
  display: inline-block;
  position: relative;
  width: 30px;
  height: 18px;
}
.lds-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 30px;
  height: 30px;
  margin-bottom: 0;
  margin-top: 0;
  margin-inline: 0;
  border: 3px solid #fff;
  border-radius: 50%;
  animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: #fff transparent transparent transparent;
}
.lds-ring div:nth-child(1) {
  animation-delay: -0.45s;
}
.lds-ring div:nth-child(2) {
  animation-delay: -0.3s;
}
.lds-ring div:nth-child(3) {
  animation-delay: -0.15s;
}
@keyframes lds-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.modal-confirm {
  color: #636363;
  width: 325px;
  font-size: 14px;
}
.modal-confirm .modal-content {
  padding: 20px;
  border-radius: 5px;
  border: none;
}
.modal-confirm .modal-header {
  border-bottom: none;
  position: relative;
}
.modal-confirm h4 {
  text-align: center;
  font-size: 26px;
  margin: 30px 0 -15px;
}
.modal-confirm .form-control,
.modal-confirm .btn {
  min-height: 40px;
  border-radius: 3px;
}
.modal-confirm .close {
  position: absolute;
  top: -5px;
  right: -5px;
}
.modal-confirm .modal-footer {
  border: none;
  text-align: center;
  border-radius: 5px;
  font-size: 13px;
}
.modal-confirm .icon-box {
  color: #fff;
  position: absolute;
  margin: 0 auto;
  left: 0;
  right: 0;
  top: -70px;
  width: 95px;
  height: 95px;
  border-radius: 50%;
  z-index: 9;
  background: #82ce34;
  padding: 15px;
  text-align: center;
  box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.1);
}
.modal-confirm .icon-box i {
  font-size: 58px;
  position: relative;
  top: 3px;
}
.modal-confirm.modal-dialog {
  margin-top: 80px;
}
.modal-confirm .btn {
  color: #fff;
  border-radius: 4px;
  background: #82ce34;
  text-decoration: none;
  transition: all 0.4s;
  line-height: normal;
  border: none;
}
.modal-confirm .btn:hover,
.modal-confirm .btn:focus {
  background: #6fb32b;
  outline: none;
}
.trigger-btn {
  display: inline-block;
  margin: 100px auto;
}
</style>
