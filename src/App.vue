<template>
  <b-container class="m-5">
    <b-row>
      <b-col>
        <b-form>
          <b-form-group
            id="file_upload_group"
            label="File upload:"
            label-for="file_upload"
            description="Upload document template"
          >
            <vue2Dropzone ref="myVueDropzone" id="file_upload" :options="dropzoneOptions"
            @vdropzone-success="onSuccessUpload">
            </vue2Dropzone>
          </b-form-group>

          <b-form-group
            id="sender_name_points_group"
            label="Sender name points (X,Y):"
            label-for="sender_name_points"
            description="Select sender name points X-Axis Y-Axis"
          >
            <p>X-Axis = {{ form.x_sender_name }}</p>
            <p>Y-Axis = {{ form.y_sender_name }}</p>
            <p>Page = {{ form.page_sender_name }}</p>
            <b-button type="button" variant="secondary"
                      @click="selectSenderName">
              Select sender name points
            </b-button>
          </b-form-group>

          <b-form-group
            id="recipient_name_points_group"
            label="Recipient name points (X,Y):"
            label-for="recipient_name_points"
            description="Select recipient name points X-Axis Y-Axis"
          >
            <p>X-Axis = {{ form.x_recipient_name }}</p>
            <p>Y-Axis = {{ form.y_recipient_name }}</p>
            <p>Page = {{ form.page_recipient_name }}</p>
            <b-button type="button" variant="secondary"
                      @click="selectRecipientName">
              Select recipient name points</b-button>
          </b-form-group>
        </b-form>
      </b-col>
      <b-col>
        <b-form-group
          id="canvas_pdf_group"
          label="Canvas pdf"
          label-for="canvas_pdf"
          description="Rendered PDF"
        >
          <p style="color: red;" v-if="axisSelectEvent !== ''">
            Please select {{ axisSelectEvent }} location</p>
          <b-row v-if="rendered" class="ml-1">
            <b-button variant="secondary" @click="page--">Prev</b-button>
            <p class="mr-1 ml-1">{{ page }}</p>
            <b-button variant="secondary" @click="page++">Next</b-button>
          </b-row>
          <canvas id="canvas_pdf" style="border:2px solid red"
                  @click="captureAxis($event)"></canvas>
        </b-form-group>
      </b-col>
    </b-row>
  </b-container>
</template>
<script>
import vue2Dropzone from 'vue2-dropzone';
import 'vue2-dropzone/dist/vue2Dropzone.min.css';

const PDFJS = require('pdfjs-dist');

export default {
  name: 'app',
  components: {
    vue2Dropzone,
  },
  watch: {
    page() {
      this.renderPdf();
    },
  },
  data() {
    return {
      dropzoneOptions: {
        url: 'https://httpbin.org/post',
        thumbnailWidth: 150,
        maxFilesize: 5,
        headers: { 'My-Awesome-Header': 'header value' },
      },
      form: {
        file_upload: '',
        x_sender_name: 0,
        y_sender_name: 0,
        page_sender_name: 0,
        x_recipient_name: 0,
        y_recipient_name: 0,
        page_recipient_name: 0,
      },
      rendered: false,
      page: 1,
      axisSelectEvent: '',
    };
  },
  methods: {
    async onSuccessUpload(file, response) {
      this.form.file_upload = response.files.file;
      this.renderPdf();
    },
    renderPdf() {
      PDFJS.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.5.207/pdf.worker.min.js';
      const loadingTask = PDFJS
        .getDocument({
          data: atob(this.form.file_upload.split('data:application/pdf;base64,')[1]),
        });
      loadingTask.promise.then((pdf) => {
        pdf.getPage(this.page).then((page) => {
          const scale = 1;
          const viewport = page.getViewport({ scale });

          // Prepare canvas using PDF page dimensions
          const canvas = document.getElementById('canvas_pdf');
          const context = canvas.getContext('2d');
          canvas.height = viewport.height;
          canvas.width = viewport.width;

          // Render PDF page into canvas context
          const renderContext = {
            canvasContext: context,
            viewport,
          };

          const renderTask = page.render(renderContext);
          renderTask.promise.then(() => {
            console.log('success render');
            this.rendered = true;
          });
        });
        this.$forceUpdate();
      });
    },
    captureAxis(e) {
      if (this.axisSelectEvent !== '') {
        if (this.axisSelectEvent === 'sender_name') {
          this.form.x_sender_name = e.offsetX;
          this.form.y_sender_name = e.offsetY;
          this.form.page_sender_name = this.page;
        } else {
          this.form.x_recipient_name = e.offsetX;
          this.form.y_recipient_name = e.offsetY;
          this.form.page_recipient_name = this.page;
        }
        this.axisSelectEvent = '';
      }
    },
    selectSenderName() {
      this.axisSelectEvent = 'sender_name';
    },
    selectRecipientName() {
      this.axisSelectEvent = 'recipient_name';
    },
  },
};
</script>

<style lang="scss">
@import 'node_modules/bootstrap/scss/bootstrap.scss';
@import 'node_modules/bootstrap-vue/src/index.scss';
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
