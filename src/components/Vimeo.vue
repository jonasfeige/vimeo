<template>
  <k-field :label="label" :required="required" :help="help" :translate="translate">
    <k-input
      type="text"
      theme="field"
      :value="value"
      @input="onInput"
    ></k-input>

    <div class="vimeo-player" v-if="iframe" ref="player">
      <div v-html="iframe"></div>
    </div>

    <k-info-field
      class="info"
      v-if="success"
      theme="positive"
      text="Alles in Ordnung"
    >
    </k-info-field>

    <k-info-field class="info" v-if="error" theme="negative" :text="error">
    </k-info-field>
  </k-field>
</template>

<script>
import vimeoRegex from "vimeo-regex";

export default {
  data() {
    return {
      error: false,
      success: false,
      iframe: null,
      settings: {
        byline: false,
        dnt: true,
        portrait: false,
        title: false,
        transparent: false,
      },
    };
  },
  props: {
    label: String,
    value: String,
    help: String,
    required: Boolean,
    translate: Boolean
  },
  computed: {
    params: function () {
      return Object.keys(this.settings)
        .map((key) => {
          return (
            encodeURIComponent(key) +
            "=" +
            encodeURIComponent(this.settings[key])
          );
        })
        .join("&");
    },
  },
  watch: {
    value() {
      if (this.value.length >= 1) {
        if (vimeoRegex().test(`${this.value}`)) {
          this.loadVideo(this.value);
        }
      } else {
        this.reset();
      }
    },
  },
  methods: {
    reset() {
      this.error = false;
      this.success = false;
      this.iframe = null;
    },
    onInput(input) {
      this.$emit("input", input);
    },
    loadVideo() {
      this.reset();
      const url = `https://vimeo.com/api/oembed.json?url=${this.value}?${this.params}`;
      fetch(url, {
        method: "GET",
        referrer: window.location.host
      })
        .then((data) => {
          if (data.ok) {
            return data.json();
          } else {
            this.reset();
            if (data.status === 404) {
              this.error = data.status + ":Video nicht gefunden";
            } else if(data.status === 403) {
              this.error = data.status + ": Zugriff verboten";
            } else {
              this.error = "Ein Fehler ist aufgetreten";
            }
          }
        })
        .then((res) => {
          /* Add referrer policy to iframe */
          let iframe = res.html
          let referrer = ' referrerpolicy="strict-origin-when-cross-origin"'
          let position = 7
          var output = [iframe.slice(0, position), referrer, iframe.slice(position)].join('');
          /* Setup iframe and success */
          this.iframe = output;
          this.success = true;
        })
        .catch((error) => {
          console.log(error.message);
        });
    },
  },
  mounted() {
    if (this.value) this.loadVideo(this.value);
  },
};
</script>

<style lang="scss">
.vimeo-player {
  width: 100%;
  overflow: hidden;
  position: relative;
  padding-bottom: 56.25%;
  border: 1px solid #ccc;
  background: white;
  margin-top: 0.5rem;
  iframe {
    position: absolute !important;
    top: 0 !important;
    left: 0 !important;
    width: 100% !important;
    height: 100% !important;
  }
}
</style>
