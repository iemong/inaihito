<template lang="pug">
    - path = "@/assets/images/"
    .home
        .first(v-if="!isFirst")
            label(for="first") 集合写真を追加してください
            input(type="file" id="first" ref="allTogether" @change="addFile")
        .second(v-if="isFirst")
            label(for="second") 来れなかった人の写真を追加してください
            input(type="file" id="second" ref="file" @change="insertFile")
        .image-wrapper(ref="imageWrapper")
        .third(v-if="isSecond")
            a(:href="downloadUrl" download="peaceful.png") 保存する
</template>

<script>
import _ from "lodash";
import html2canvas from "html2canvas";

const URL = "https://kiyoshidainagon-node-red-app.mybluemix.net/vr/recognize";

export default {
  name: "home",
  data() {
    return {
      width: 0,
      height: 0,
      left: 0,
      top: 0,
      isFirst: false,
      isSecond: false,
      downloadUrl: null
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
      return ` circle(${this.length / 2 + 20}px at ${this.center.x}px ${
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
      this.$refs.imageWrapper.style.width = `${$image.width}px`;

      this.isFirst = true;
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

      $image.classList.add("trim-image");
      $image.style.clipPath = this.position;
      $image.style.top = `${(this.center.y - this.height / 2 - 30) * -1}px`;
      $image.style.left = `${(this.center.x + this.width / 2 + 30) * -1 +
        this.$refs.imageWrapper.clientWidth}px`;
      $image.style.transformOrigin = `${this.center.x + this.width / 2}px ${this
        .center.y -
        this.height / 2}px`;
      $image.style.transform = "scale(0.5)";
      this.$refs.imageWrapper.appendChild($image);

      const canvas = await html2canvas(this.$refs.imageWrapper);
      this.downloadUrl = canvas.toDataURL("image/png");
      this.isSecond = true;
    }
  }
};
</script>

<style lang="sass">
    .image-wrapper
        position: relative
        width: 800px
        text-align: left
    .trim-image
        position: absolute
</style>
