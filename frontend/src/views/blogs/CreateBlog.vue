<template>
  <div class="container is-widescreen">
    <section class="section" v-if="error">
      <div class="container is-widescreen">
        <div class="notification is-danger">
          <!-- <%= error.code + ': ' + error.sqlMessage %> -->
          <!---->
          {{ error }}
        </div>
      </div>
    </section>
    <section class="hero">
      <div class="hero-body">
        <p class="title">Create new Blog</p>
      </div>
    </section>
    <section class="px-6">
      <input
        class="mb-5"
        multiple
        type="file"
        accept="image/png, image/jpeg, image/webp"
        @change="selectImages"
      />

      <div v-if="images" class="columns is-multiline">
        <div
          v-for="(image, index) in images"
          :key="image.id"
          class="column is-one-quarter"
        >
          <div class="card">
            <div class="card-image">
              <figure class="image is-4by3">
                <img :src="showSelectImage(image)" alt="Placeholder image" />
              </figure>
            </div>
            <footer class="card-footer">
              <a
                @click="deleteSelectImage(index)"
                class="card-footer-item has-text-danger"
                >Delete</a
              >
            </footer>
          </div>
        </div>
      </div>

      <div class="field mt-5">
        <label class="label">Title</label>
        <div class="control">
          <input
            v-model="$v.titleBlog.$model"
            :class="{ 'is-danger': $v.titleBlog.$error }"
            class="input"
            type="text"
            placeholder="Title"
          />
        </div>
        <template v-if="$v.titleBlog.$error">
          <p class="help is-danger" v-if="!$v.titleBlog.required">
            This field is required
          </p>
          <p
            class="help is-danger"
            v-if="!$v.titleBlog.minLength && $v.titleBlog.required"
          >
            too short
          </p>
          <p
            class="help is-danger"
            v-if="
              !$v.titleBlog.maxLength &&
              $v.titleBlog.required &&
              $v.titleBlog.minLength
            "
          >
            too long
          </p>
        </template>
      </div>

      <div class="field">
        <label class="label">Content</label>
        <div class="control">
          <input
            v-model="$v.contentBlog.$model"
            :class="{ 'is-danger': $v.contentBlog.$error }"
            class="input"
            type="text"
            placeholder="Content"
          />
        </div>
        <template v-if="$v.contentBlog.$error">
          <p class="help is-danger" v-if="!$v.contentBlog.required">
            This field is required
          </p>
          <p
            class="help is-danger"
            v-if="!$v.contentBlog.minLength && $v.contentBlog.required"
          >
            too short
          </p>
        </template>
      </div>

      <div class="field">
        <label class="label">Reference</label>
        <div class="control">
          <input
            v-model="$v.referenceBlog.$model"
            :class="{ 'is-danger': $v.referenceBlog.$error }"
            class="input"
            type="text"
            placeholder="Reference"
          />
        </div>
        <template v-if="$v.referenceBlog.$error">
          <p class="help is-danger" v-if="!$v.referenceBlog.required">
            This field is required
          </p>
          <p
            class="help is-danger"
            v-if="!$v.referenceBlog.url && $v.referenceBlog.required"
          >
            only url
          </p>
        </template>
      </div>

      <div class="control mb-3">
        <label class="radio">
          <input v-model="statusBlog" type="radio" name="answer" value="01" />
          Private
        </label>
        <label class="radio">
          <input v-model="statusBlog" type="radio" name="answer" value="02" />
          Public
        </label>
      </div>

      <div class="field">
        <div class="control">
          <label class="checkbox">
            <input v-model="pinnedBlog" type="checkbox" />
            Pinned
          </label>
        </div>
      </div>

      <div class="control">
        <div class="columns">
          <div class="column is-6">
            <div class="field">
              <label class="label">วันที่โพสท์</label>
              <div class="control">
              <input
                v-model="$v.start_date.$model"
                type="date"
                class="input"
                placeholder="Textarea"
              />
              </div>
            </div>
            <template v-if="$v.start_date.$error">
              <p class="help is-danger" v-if="!$v.start_date.start_date">
                This field is required
              </p>
              <p class="help is-danger" v-if="!$v.start_date.checkdate">
                invalid date
              </p>
            </template>
          </div>
          <div class="column is-6">
            <div class="field">
              <label class="label">วันสิ้นสุดโพส</label>
              <input
                v-model="$v.end_date.$model"
                type="date"
                class="input"
                placeholder="Textarea"
              />
            </div>
            <template v-if="$v.end_date.$error">
              <p class="help is-danger" v-if="!$v.end_date.end_date">
                This field is required
              </p>
              <p class="help is-danger" v-if="!$v.end_date.checkdate">
                invalid date
              </p>
            </template>
          </div>
        </div>
      </div>

      <div class="field is-grouped mt-5">
        <div class="control">
          <button @click="submitBlog" class="button is-link">Submit</button>
        </div>
        <div class="control">
          <button @click="$router.go(-1)" class="button is-link is-light">
            Cancel
          </button>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import axios from "axios";
import {
  required,
  email,
  minLength,
  maxLength,
  sameAs,
  url,
} from "vuelidate/lib/validators";

function start_date() {
  if (this.end_date == "" && this.start_date == "") {
    return true;
  } else if (this.end_date !== "" && this.start_date !== "") {
    return true;
  } else if (this.end_date == "" && this.start_date !== "") {
    this.$v.end_date.$touch();
    return true;
  } else {
    return false;
  }
}
function end_date() {
  if (this.end_date == "" && this.start_date == "") {
    return true;
  } else if (this.end_date !== "" && this.start_date !== "") {
    return true;
  } else if (this.end_date !== "" && this.start_date == "") {
    this.$v.start_date.$touch();
    return true;
  } else {
    return false;
  }
}
function checkdate() {
  if (this.end_date !== "" && this.start_date !== "") {
    if (this.start_date < this.end_date) {
      return true;
    } else {
      return false;
    }

  }else{
    return true
  }
}

export default {
  data() {
    return {
      blog: {},
      error: null,
      images: [], // array of image
      titleBlog: "",
      contentBlog: "",
      pinnedBlog: false,
      statusBlog: "01",
      referenceBlog: "",
      start_date: "",
      end_date: "",
      keep: "",
    };
  },
  validations: {
    titleBlog: {
      required: required,
      minLength: minLength(10),
      maxLegth: maxLength(25),
    },
    contentBlog: {
      required: required,
      minLength: minLength(50),
    },
    referenceBlog: {
      required: required,
      url: url,
    },
    start_date: {
      start_date: start_date,
      checkdate: checkdate,
    },
    end_date: {
      end_date: end_date,
      checkdate: checkdate,
    },
  },
  methods: {
    selectImages(event) {
      let img = event.target.files
      if(img[0].size > 1000000){
        alert("file size must be < 1mb")
      }else{
        this.images = img;
      }
      console.log(this.images)
    },
    showSelectImage(image) {
      // for preview only
      return URL.createObjectURL(image);
    },
    deleteSelectImage(index) {
      console.log(this.images);
      this.images = Array.from(this.images);
      this.images.splice(index, 1);
    },
    submitBlog() {
      this.$v.$touch();
      let formData = new FormData();
      formData.append("title", this.titleBlog);
      formData.append("content", this.contentBlog);
      formData.append("pinned", this.pinnedBlog ? 1 : 0);
      formData.append(
        "status",
        (this.statusBlog = "01" ? "status_private" : "status_public")
      );
      formData.append("reference", this.referenceBlog);
      formData.append("start_date", this.start_date);
      formData.append("end_date", this.end_date);
      this.images.forEach((image) => {
        formData.append("myImage", image);
      });
      console.log("form", formData);

      // Note ***************
      // ตอนเรายิง Postmant จะใช้ fromData
      // ตอนยิงหลาย ๆ รูปพร้อมกันใน Postman จะเป็นแบบนี้

      // title   | "This is a title of blog"
      // comment | "comment in blog"
      // ...
      // myImage | [select file 1]
      // myImage | [select file 2]
      // myImage | [select file 3]

      // จะสังเกตุว่าใช้ myImage เป็น key เดียวกัน เลยต้องเอามา loop forEach
      // พอไปฝั่ง backend มันจะจัด file ให้เป็น Array เพื่อเอาไปใช้งานต่อได้

      axios
        .post("http://localhost:3000/blogs", formData)
        .then((res) => this.$router.push({ name: "home" }))
        .catch((e) => console.log(e.response.data));
    },
  },
};
</script>

<style>
</style>