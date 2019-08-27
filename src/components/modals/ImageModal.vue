<template>
  <modal-inner aria-label="Insert image">
    <div class="modal__content">
      <div v-if="url === null" class="image_dropzone">
        <div>Drop your Image here</div>
        <div>- or -</div>
        <div><input type="file" name="fileToUpload" id="fileToUpload" v-on:change="handleFileUpload($event.target.name, $event.target.files)"></div>
      </div>
      <img v-else v-bind:src="url" />
      <div class="image_modal__input">
        <label>Image Width:</label>
        <input v-model="width"/>
      </div>
      <div class="image_modal__input">
        <label>Image Caption:</label>
        <input v-model="caption"/>
      </div>
    </div>
    <div class="modal__button-bar">
      <button class="button" @click="reject()">Cancel</button>
      <button v-bind:disabled="url === null" class="button button--resolve" @click="resolve">Ok</button>
    </div>
  </modal-inner>
</template>

<style lang="scss">
  .image_dropzone {
    height: 400px;
    width: 100%;
    border: 2px dashed black;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 10px;
    flex-direction: column;
  }

  .image_modal__input {
    margin-top: 15px;

    label {
      width: 250px;
    }
  }
</style>

<script>
import modalTemplate from './common/modalTemplate';
import MenuEntry from '../menus/common/MenuEntry';
import utils from '../../services/utils';

const {
  origin,
} = utils.queryParams;

export default modalTemplate({
  components: {
    MenuEntry,
  },
  data: () => ({
    url: null,
    width: '',
    caption: '',
  }),
  methods: {
    resolve(evt) {
      evt.preventDefault(); // Fixes https://github.com/benweet/stackedit/issues/1503
      if (!this.url) {
        this.setError('url');
      } else {
        const { callback } = this.config;
        this.config.resolve();
        debugger;
        callback(this.url, this.width, this.caption);
      }
    },
    reject() {
      const { callback } = this.config;
      this.config.reject();
      callback(null);
    },
    handleFileUpload(fieldName, fileList) {
      console.log('origin', origin);
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
      }, 'http://localhost:8080');
    },
  },
});
</script>
