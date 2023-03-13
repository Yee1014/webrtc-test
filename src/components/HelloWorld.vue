<script setup lang="ts">
import { onMounted, ref } from 'vue'

const title = ref('WebRTC 拍照')

// 想要获取一个最接近 1280x720 的相机分辨率
const constraints = {
  audio: false,
  video: {
    // 帧率
    frameRate: {ideal: 30, max: 45},
    width: 'auto',
    height: 'auto',
    // width: {min: 640, ideal: 1280, max: 1920},
    // height: {min: 360, ideal: 720, max: 1080},
    // 前置摄像头（如果有的话）
    facingMode: 'user',
    // 强制使用后置摄像头
    // facingMode: {exact: 'environment'},
  },
}

const attachVideo = (stream: MediaStream) => {
  const video = document.querySelector('video')
  if (!video) return
  // 旧的浏览器可能没有 srcObject
  if ('srcObject' in video) {
    video.srcObject = stream
  } else {
    // 防止在新的浏览器里使用它，应为它已经不再支持了
    video.src = window.URL.createObjectURL(stream)
  }
  video!.onloadedmetadata = function (e) {
    video!.play()
  }
}

const getCamera = () => {

  navigator.mediaDevices.getUserMedia(constraints)
    .then(function (mediaStream) {
      attachVideo(mediaStream)
    })
    .catch(function (err) {
      console.log(err.name + ': ' + err.message)
    })
}

const requestMedia = async (displayMediaOptions = {...constraints}) => {

  try {
    const captureStream = await navigator.mediaDevices.getDisplayMedia(displayMediaOptions)
    attachVideo(captureStream)
  } catch (err) {
    console.error('Error: ' + err)
  }
}

const deviceList = ref<MediaDeviceInfo[]>([])

const getEnumerateDevices = async () => {
  try {
    deviceList.value = await navigator.mediaDevices.enumerateDevices()
  } catch (e) {
    console.log(e)
  }
}

const getSupportedConstraints = () => {
  console.log(navigator.mediaDevices.getSupportedConstraints())
}

onMounted(() => {
  getSupportedConstraints()
  getEnumerateDevices()
})

</script>

<template>
  <h2>{{ title }}</h2>
  <ul class="d-list">
    <li v-for="d in deviceList" :key="d.deviceId">
      {{
        d.kind + ': ' + d.label +
        ' id = ' + d.deviceId
      }}
    </li>
  </ul>
  <button @click="requestMedia">添加窗口</button>
  <button @click="getCamera">添加摄像头</button>
  <div class="container">
    <video id="video">视频流目前不可用。</video>
  </div>
</template>

<style scoped>
.d-list {
  color: #888;
}

.d-list li {
  text-align: left;
}
</style>
