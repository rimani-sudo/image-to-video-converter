<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image to Video Converter</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      padding: 2rem;
      background: #f7f7f7;
    }
    h1 {
      color: #333;
    }
    input, button {
      margin: 10px;
      padding: 10px;
      font-size: 16px;
    }
    #videoResult {
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <h1>Image to Video Converter (Free)</h1>
  <input type="file" id="images" multiple accept="image/*"><br>
  <input type="number" id="duration" placeholder="Duration per image (sec)" value="2" min="1"><br>
  <button onclick="convertImagesToVideo()">Convert to Video</button>

  <div id="videoResult"></div>

  <script src="https://cdn.jsdelivr.net/npm/@ffmpeg/ffmpeg@0.12.6/dist/ffmpeg.min.js"></script>
  <script>
    const { createFFmpeg, fetchFile } = FFmpeg;
    const ffmpeg = createFFmpeg({ log: true });

    async function convertImagesToVideo() {
      const files = document.getElementById('images').files;
      const duration = document.getElementById('duration').value || 2;
      if (files.length < 1) {
        alert("Upload at least one image");
        return;
      }

      document.getElementById("videoResult").innerText = "Processing...";

      await ffmpeg.load();

      // Load images into FFmpeg FS
      for (let i = 0; i < files.length; i++) {
        ffmpeg.FS('writeFile', `img${i}.jpg`, await fetchFile(files[i]));
      }

      // Create input.txt for ffmpeg
      let inputTxt = "";
      for (let i = 0; i < files.length; i++) {
        inputTxt += `file 'img${i}.jpg'\nduration ${duration}\n`;
      }
      inputTxt += `file 'img${files.length - 1}.jpg'\n`; // Repeat last image
      ffmpeg.FS('writeFile', 'input.txt', inputTxt);

      // Run ffmpeg to generate video
      await ffmpeg.run(
        '-f', 'concat',
        '-safe', '0',
        '-i', 'input.txt',
        '-vsync', 'vfr',
        '-pix_fmt', 'yuv420p',
        'output.mp4'
      );

      const data = ffmpeg.FS('readFile', 'output.mp4');
      const videoBlob = new Blob([data.buffer], { type: 'video/mp4' });
      const videoUrl = URL.createObjectURL(videoBlob);

      // Show video
      document.getElementById("videoResult").innerHTML = `
        <video controls width="320" src="${videoUrl}"></video><br>
        <a href="${videoUrl}" download="output.mp4">Download Video</a>
      `;
    }
  </script>
</body>
</html>
