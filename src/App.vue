<template>
  <div>
    <video width="800" ref="viewWindow" autoplay></video>
  </div>
  <select @change="changeCamera($event)">
    <option disabled value="">Please select one</option>
    <option value="environment">Back</option>
    <option value="user">Front</option>
  </select>
  <button @click="takePhoto">Take Photo</button>
</template>

<script>
export default {
  name: "App",
  data: function () {
    return {
      imageCapture: null,
      video: null,
      cameraSource: "environment",
    };
  },
  mounted() {
    this.refreshVideoFeed();
  },
  methods: {
    refreshVideoFeed() {
      navigator.mediaDevices
        .getUserMedia({
          video: {
            facingMode: this.cameraSource,
          },
        })
        .then((mediaStream) => {
          this.$refs.viewWindow.srcObject = mediaStream;
          const track = mediaStream.getVideoTracks()[0];
          this.imageCapture = new ImageCapture(track);
        })
        .catch((error) => console.log(error));
    },
    changeCamera(event) {
      this.cameraSource = event.target.value;
      this.refreshVideoFeed();
    },
    takePhoto() {
      this.imageCapture
        .takePhoto()
        .then((blob) => {
          this.downloadBlob(blob);
        })
        .catch((error) => console.log(error));
    },
    downloadBlob(blob, name = "photo.jpeg") {
      // Convert your blob into a Blob URL (a special url that points to an object in the browser's memory)
      const blobUrl = URL.createObjectURL(blob);

      // Create a link element
      const link = document.createElement("a");

      // Set link's href to point to the Blob URL
      link.href = blobUrl;
      link.download = name;

      // Append link to the body
      document.body.appendChild(link);

      // Dispatch click event on the link
      // This is necessary as link.click() does not work on the latest firefox
      link.dispatchEvent(
        new MouseEvent("click", {
          bubbles: true,
          cancelable: true,
          view: window,
        })
      );

      // Remove link from body
      document.body.removeChild(link);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
