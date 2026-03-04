# vscodeBackgroundProfile

## Overview

This is an example file for creating a `setting.json` file for vscode to set a background picture.

## setting.json

```json
{
"background.fullscreen": {
    "useFront": true,
    "style": {
      "background-position": "100% 100%",
      "background-size": "auto",
      "opacity": 0.6
    },
    "styles": [{}, {}, {}],
    // `images` supports online and local images, as well as folders.
    "images": [
      "D:\\pictures\\FD6sk4xaMAInI7I.jpg"
      // online images, only `https` is allowed.
      //"https://hostname/online.jpg",
      // local images
      //"file:///local/path/img.jpeg",
      //"/home/xie/downloads/img.gif",
      //"C:/Users/xie/img.bmp",
      //"D:\\downloads\\images\\img.webp",
      // local folders
      //"/home/xie/images",
      // data URL
      //"data:image/*;base64,<base64-data>"
    ],
    "interval": 0,
    "random": false
  }
}
```