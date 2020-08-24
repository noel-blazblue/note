

```js
// 根据自定义最大宽度缩小图片
function resizeImg(width, height) {
  let imgW = style.imgW1
  let cHeight = null
  let cWidth = null
  if (width > imgW) {
    cHeight = Math.floor(imgW/width*height)
    cWidth = imgW
  } else {
    cHeight = height
    cWidth = width
  }
  return [cWidth, cHeight]
}

// 从相册发送图片
const sendImg = async() => {
  const options = {
    title: 'Select Avatar',
    customButtons: [{ name: 'fb', title: 'Choose Photo from Facebook' }],
    storageOptions: {
      skipBackup: true,
      path: 'images',
    },
  }
  let attr 
  attr = await ImagePicker.launchImageLibrary(options, (response) => {
    if (response.didCancel) {
      console.log('user cancel select photo')
    } else if (response.error) {
      console.log('ImagePicker err', response.error)
    } else if (response.customButton) {
      console.log('user tapped custom button', response.customButton)
    } else {
      let resArr = util.resizeImg(response.width, response.height)
      attr = [
        { uri: 'data:image/jpeg;base64,' + response.data },
        { width: resArr[0], height: resArr[1] }
      ]
      return attr
    }
  })
  console.log('button', attr)
  return attr
}

// 从相机发送图片
const sendImgFromCamera = () => {
  const options = {
    title: 'Select Avatar',
    customButtons: [{ name: 'fb', title: 'Choose Photo from Facebook' }],
    storageOptions: {
      skipBackup: true,
      path: 'images',
    },
  }
  ImagePicker.launchCamera(options, (response) => {
    console.log('responsr', response)
    if (response.didCancel) {
      console.log('user cancel select photo')
    } else if (response.error) {
      console.log('ImagePicker err', response.error)
    } else if (response.customButton) {
      console.log('user tapped custom button', response.customButton)
    } else {
      let resArr = util.resizeImg(response.width, response.height)
      let attr = [
        { uri: 'data:image/jpeg;base64,' + response.data },
        { width: resArr[0], height: resArr[1] }
      ]
      return attr
    }
  })
}
```

