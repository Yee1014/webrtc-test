<template>
  <div class="contentarea">
    <h3>MDN——navigator.mediaDevices.getUserMedia(): 静态照片拍摄演示</h3>
    <p>
      此示例演示了如何使用内置的网络摄像头来获取媒体流，并从中获取图像，以使用该图像来创建一个 PNG 图像。
    </p>
    <div class="camera">
      <video id="video">视频流目前不可用。</video>
      <button id="startbutton">拍摄照片</button>
    </div>
    <canvas id="canvas"></canvas>
    <div class="output">
      <img id="photo" alt="捕获的图像会显示在这里。"/>
      <button id="downloadbutton">下载照片</button>
    </div>
    <p>
      访问我们的文章：<a
      href="https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API/Taking_still_photos">
      使用 getUserMedia() 拍摄静态照片</a
    >以详细了解此处使用的技术。
    </p>
  </div>
</template>

<script setup lang="ts">
// The width and height of the captured photo. We will set the
// width to the value defined here, but the height will be
// calculated based on the aspect ratio of the input stream.

import { onMounted } from 'vue'

const width = 320 // We will scale the photo width to this
let height = 0 // This will be computed based on the input stream

// |streaming| indicates whether or not we're currently streaming
// video from the camera. Obviously, we start at false.

let streaming = false

// The various HTML elements we need to configure or control. These
// will be set by the startup() function.

let video: HTMLVideoElement | null = null
let canvas: HTMLCanvasElement | null = null
let photo: HTMLImageElement | null = null
let startbutton: HTMLButtonElement | null = null
let downloadbutton: HTMLButtonElement | null = null

/**
 * iframe中不支持
 */
function showViewLiveResultButton() {
  if (window.self !== window.top) {
    // Ensure that if our document is in a frame, we get the user
    // to first open it in its own tab or window. Otherwise, it
    // won't be able to request permission for camera access.
    document.querySelector('.contentarea')?.remove()
    const button = document.createElement('button')
    button.textContent = '查看以上示例代码的实时演示'
    document.body.append(button)
    button.addEventListener('click', () => window.open(location.href))
    return true
  }
  return false
}

function startup() {
  if (showViewLiveResultButton()) {
    return
  }
  video = document.getElementById('video') as HTMLVideoElement
  canvas = document.getElementById('canvas') as HTMLCanvasElement
  photo = document.getElementById('photo') as HTMLImageElement
  startbutton = document.getElementById('startbutton') as HTMLButtonElement
  downloadbutton = document.getElementById('downloadbutton') as HTMLButtonElement

  /**
   * 请求获取摄像头，无音频
   */
  navigator.mediaDevices.getUserMedia({video: true, audio: false})
    .then((stream) => {
      video!.srcObject = stream
      video!.play()
    })
    .catch((err) => {
      console.error(`An error occurred: ${err}`)
    })

  /**
   * 监听视频是否可播放
   */
  video.addEventListener('canplay', (ev) => {
    if (!streaming) {
      height = video!.videoHeight / (video!.videoWidth / width)

      // Firefox currently has a bug where the height can't be read from
      // the video, so we will make assumptions if this happens.

      if (isNaN(height)) {
        height = width / (4 / 3)
      }

      video!.setAttribute('width', width + '')
      video!.setAttribute('height', height + '')
      canvas!.setAttribute('width', width + '')
      canvas!.setAttribute('height', height + '')
      streaming = true
    }
  }, false)

  startbutton.addEventListener('click', (ev) => {
    takepicture()
    ev.preventDefault()
  }, false)

  downloadbutton.addEventListener('click', (ev) => {
    downloadImg()
    ev.preventDefault()
  }, false)

  clearphoto()
}

// Fill the photo with an indication that none has been
// captured.

function clearphoto() {
  const context = canvas!.getContext('2d')
  context!.fillStyle = '#AAA'
  context!.fillRect(0, 0, canvas!.width, canvas!.height)

  const data = canvas!.toDataURL('image/png')
  photo!.setAttribute('src', data)
}

function downloadImg() {
  const aLink = document.createElement('a')
  aLink.download = 'phone.png'
  aLink.href = photo?.getAttribute('src') as string
  aLink.click()
}

// Capture a photo by fetching the current contents of the video
// and drawing it into a canvas, then converting that to a PNG
// format data URL. By drawing it on an offscreen canvas and then
// drawing that to the screen, we can change its size and/or apply
// other changes before drawing it.

function takepicture() {
  const context = canvas!.getContext('2d')
  if (width && height) {
    canvas!.width = width
    canvas!.height = height
    context!.drawImage(video!, 0, 0, width, height)

    // 展示图片
    const data = canvas!.toDataURL('image/png')
    photo!.setAttribute('src', data)

    canvas!.toBlob(
      (blob) => {
        if (blob) {
          // 上传到服务器
          /*
          const formData = new FormData()
          formData.append('file', blob)
          axios({
            method: 'post',
            url: 'your_url',
            data: formData,
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          })
          */
        }
      },
      'image/png',
      1, // 质量高给后端服务分析
    )
  } else {
    clearphoto()
  }
}

// Set up our event listener to run the startup process
// once loading is complete.
// window.addEventListener('load', startup, false)
onMounted(() => {
  startup()
})
</script>

<style scoped>
#video {
  border: 1px solid black;
  box-shadow: 2px 2px 3px black;
  width: 320px;
  height: 240px;
}

#photo {
  border: 1px solid black;
  box-shadow: 2px 2px 3px black;
  width: 320px;
  height: 240px;
}

#canvas {
  display: none;
}

.camera {
  width: 340px;
  display: inline-block;
}

.output {
  width: 340px;
  display: inline-block;
  vertical-align: top;
}

#startbutton,
#downloadbutton {
  display: block;
  position: relative;
  margin-left: auto;
  margin-right: auto;
  bottom: 32px;
  background-color: rgba(0, 150, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.7);
  box-shadow: 0 0 1px 2px rgba(0, 0, 0, 0.2);
  font-size: 14px;
  font-family: "Lucida Grande", "Arial", sans-serif;
  color: rgba(255, 255, 255, 1);
}

.contentarea {
  margin: 0 auto;
  font-size: 16px;
  font-family: "Lucida Grande", "Arial", sans-serif;
  width: 760px;
}
</style>
