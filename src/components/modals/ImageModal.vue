<template>
  <modal-inner aria-label="Insert image">
    <div class="modal__content">
      <div class="selected_image" v-if="url !== null">
        Selected Image:
        <img v-bind:src="url" />
      </div>
      <div class="image_dropzone">
        <div class="image_modal__input">
          <label>Upload an Image:</label>
          <input type="file" name="fileToUpload" id="fileToUpload" v-on:change="handleFileUpload($event.target.name, $event.target.files)"></div>
          <div style="font-weight: bold;">or</div>
          <div class="image_modal_selecet" v-on:click="handleClickEnableImageSelect">Select an image</div>
          <div v-if="isImageSelectionActive" class="image_select">
            <div v-for="image in images" v-bind:key="image.title" v-on:click="() => handleClickImage(image.url)" v-bind:class="{ active: image.url === url }">
              <img v-bind:src="image.url + '?w=200'" />
              <div class="title">{{ image.title }}</div>
            </div>
          </div>
      </div>
      <div class="image_modal__input">
        <label>Image Width:</label>
        <input v-model="width" type="number"/>
      </div>
      <div class="image_modal__input">
        <label>Image Caption:</label>
        <input v-model="caption" placeholder="Caption displayed under the image"/>
      </div>
      <div class="image_modal__input">
        <label>Alignment:</label>
        <select v-model="alignment">
          <option value="left">Left</option>
          <option value="center">Center</option>
          <option value="right">Right</option>
        </select>
      </div>
      <div class="image_modal__input">
        <label>Image Alt:</label>
        <input v-model="imageAlt" placeholder="Alternate text for the image, if the image cannot be displayed"/>
      </div>
    </div>
    <div class="modal__button-bar">
      <button class="button" @click="reject()">Cancel</button>
      <button v-bind:disabled="url === null" class="button button--resolve" @click="resolve">Ok</button>
    </div>
  </modal-inner>
</template>

<style lang="scss">
  .image_modal__input {
    margin-top: 15px;

    label {
      width: 250px;
    }

    input {
      width: 100%;
      border-radius: 4px;
    }

    select {
      width: 100%;
    }
  }

  .image_modal_selecet {
    color: #fb6222;
    font-size: 16px;
    cursor: pointer;
    font-weight: 500;

    &:hover {
      font-weight: 700;
      text-decoration: underline;
    }
  }

  .image_select {
    height: 400px;
    width: 100%;
    border: 1px solid #fdfdfd;
    background: white;
    border-radius: 2px;
    display: grid;
    overflow-y: scroll;
    grid-template-columns: repeat(4, 1fr);
    grid-gap: 15px;

    .image_element {
      border: 1px solid black;

      .title {
        font-size: 11px;
        color: black;
        padding: 15px;
      }
    }
  }

  .selected_image {
    max-width: 250px;
    margin: 0 auto;
  }
</style>

<script>
import modalTemplate from './common/modalTemplate';
import MenuEntry from '../menus/common/MenuEntry';

export default modalTemplate({
  components: {
    MenuEntry,
  },
  data: () => ({
    url: null,
    width: 1200,
    caption: '',
    alignment: 'center',
    imageAlt: '',
    images: [],
    isImageSelectionActive: false,
  }),
  created() {
    window.addEventListener('message', ({ data: { type, payload } }) => {
      if (type === 'responseImages') {
        this.images = payload.images;
      }
    }, false);

    window.parent.postMessage({ type: 'requestImages', payload: {} }, '*');
  },
  methods: {
    resolve(evt) {
      evt.preventDefault(); // Fixes https://github.com/benweet/stackedit/issues/1503
      if (!this.url) {
        this.setError('url');
      } else {
        const { callback } = this.config;
        this.config.resolve();
        callback(this.url, this.width, this.caption, this.alignment, this.imageAlt);
      }
    },
    reject() {
      const { callback } = this.config;
      this.config.reject();
      callback(null);
    },
    handleClickEnableImageSelect() {
      this.isImageSelectionActive = !this.isImageSelectionActive;
    },
    handleClickImage(imageUrl) {
      this.url = imageUrl;
      this.isImageSelectionActive = false;
    },
    handleFileUpload(fieldName, fileList) {
      if (!fileList.length) return;

      const messageEventHandler = ({ data: { type, payload } }) => {
        if (type === 'imageUploaded') {
          this.url = payload.url;
        }
      };

      window.addEventListener('message', messageEventHandler, false);

      window.parent.postMessage({
        type: 'imageUpload',
        payload: {
          image: fileList[0],
          origin: window.origin,
        },
      }, '*');
    },
  },
});
</script>
