<template>
  <modal-inner aria-label="Insert image">
    <div class="modal__content">
      <div v-if="url === null" class="image_dropzone">
        <div class="image_modal__input">
          <label>Image:</label>
          <input type="file" name="fileToUpload" id="fileToUpload" v-on:change="handleFileUpload($event.target.name, $event.target.files)"></div>
      </div>
      <img v-else v-bind:src="url" />
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
  }),
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
