<template lang="pug">
    - path = "@/assets/images/"
    .home
        input(type="file" ref="allTogether" @change="addFile")
        input(type="file" ref="file" @change="insertFile")
        .image-wrapper(ref="imageWrapper")

</template>

<script>
import _ from "lodash";

const URL = "https://kiyoshidainagon-node-red-app.mybluemix.net/vr/recognize";

export default {
  name: "home",
  data() {
    return {
      width: 0,
      height: 0,
      left: 0,
      top: 0
    };
  },
  computed: {
    length() {
      return _.max([this.height, this.width]);
    },
    center() {
      return {
        x: this.left + this.width / 2,
        y: this.top + this.height / 2
      };
    },
    position() {
      return `circle(${this.length / 2}px at ${this.center.x}px ${
        this.center.y
      }px)`;
    }
  },
  methods: {
    loadFile(file) {
      return new Promise(resolve => {
        const reader = new FileReader();
        reader.onload = () => {
          resolve(reader.result);
        };
        reader.readAsDataURL(file);
      });
    },
    loadImage(data) {
      return new Promise(resolve => {
        const image = new Image();
        image.onload = () => {
          resolve(image);
        };
        image.src = data;
      });
    },
    addFile: async function(evt) {
      const files = evt.target.files;
      const data = await this.loadFile(files[0]);
      const $image = await this.loadImage(data);
      this.$refs.imageWrapper.appendChild($image);
    },
    insertFile: async function(evt) {
      const files = evt.target.files;
      const data = await this.loadFile(files[0]);
      const $image = await this.loadImage(data);

      const base64 = data.replace(/^.*,/, "");
      const response = await fetch(URL, { method: "POST", body: base64 });

      const json = await response.json();
      this.width = json.face_location.width;
      this.height = json.face_location.height;
      this.left = json.face_location.left;
      this.top = json.face_location.top;

      $image.style.clipPath = this.position;
      this.$refs.imageWrapper.appendChild($image);
    }
  }
};
</script>

<style lang="sass">
    .image-wrapper
        position: relative
        width: 800px
        text-align: left
</style>
