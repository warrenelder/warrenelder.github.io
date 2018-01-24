# hexo-image-figure-caption

wrap images in figures and add caption -- forked from wayou's [hexo-image-caption](https://github.com/wayou/hexo-image-caption)

[![build status](https://secure.travis-ci.org/wayou/hexo-image-caption.svg)](http://travis-ci.org/wayou/hexo-image-caption)
[![dependency status](https://david-dm.org/wayou/hexo-image-caption.svg)](https://david-dm.org/wayou/hexo-image-caption)

## Installation

```
npm install --save hexo-image-figure-caption
```

## Usage
adding following section to your hexo site `_config.yml` file to enable and config plugin.
```yml
# add caption for iamges
image_caption:
  enable: true #false to disable
  class_name: #if you wanna customize the style for the caption,you can assign a class name, default is 'image-caption'
```

Your images will be wrapped in a `figure` and the alt text will be placed in a `figcaption`. This way you have a bit more control over the layout and can keep the caption with the figure as you lay things out, e.g. when you want to put several images side-by-side.

## Credits
forked from [wayou/hexo-image-caption](https://github.com/wayou/hexo-image-caption)

## License

MIT
