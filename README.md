# learnWebpack

static module bundler for moder javascript application. recursivly check the dependecies graph that include your every module and then packages them into one or more bundles.

* earlier webapp will have multiple js and css  and other assets import. And we need to ***maintain them in correct order to fullfill dependencies.***
* Installing webpack
```
npm i -D "webpack webpack-cli
```
* By Default webpack create **dist** folder for output and create main.js file with bundled data.
* By Default starting point is ./src/index.js
  ```
  npm webpack
  npx webpack  --stats detailed    -> this give more detail while compiling and bundling the file
  ```
### Basic custom webpack config file
* Default -> rootFolder/webpack.config.js
```js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path:  path.resolve(__dirname, './dist')
    },
    mode: 'none'
};
```
### Asset Module - webpack 5 feature
* we can now process plain file , images, fonts etc without using any additional dependencies
* Four types of asset Module
  * asset/resource  -> use to import large imageor font file -> put file in output directory and export url of file
  * asset/inline    -> inline the file in bundle as data uri -> use for small file like svg -> create base64 representation and inject it in js bundle
  * asset           -> webpack will choose to use as resource or inline. by default it is based of file size -> 8kb
  * asset/source    -> to bring plain text file as stream. Import source code of file as it is and inject in js bundle

* If we are importing any file in js, webpack will check its rule object, to see, if there is any rule to import or not.
* If it does not find any rule for the given file, it will throw error.
```html
<!doctype html>

<html>
<head>
    <title>Hello world</title>
    <meta charset="utf-8">
</head>
<body>
    <script src="./dist/bundle.js"></script>
</body>
</html>
```
```js
//Main file -> entry file (index.js)
import helloWorld from './hello-world.js';
import addImage from './add-image.js';

helloWorld();
addImage();
///////////////////////////////////
function helloWorld() {
    console.log('Hello world');
}

export default helloWorld;
////////////////////////////////////
import Kiwi from './kiwi.jpg';           // file is present in same folder
import altText from './altText.txt';

function addImage() {
    // Inside this function I will create an img dom element, specify an alt , width , and src properties.
    const img = document.createElement('img');
    img.alt = altText;          // importing text from another plain file
    img.width = 300;
    img.src = Kiwi;
    const body = document.querySelector('body');
    body.appendChild(img);
}

export default addImage;
////////////////////////////////////////
altText.txt       -> plain text file. below is its content
Kiwi alt text
```
* webpack.config.js
```js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist'),
        publicPath: 'dist/'
    },
    mode: 'none',
    module: {
        rules: [
            {
                test: /\.(png|jpg)$/,
                type: 'asset',
                parser: {                          // changing size creiteria to choose asset/resource or asset/inline
                    dataUrlCondition: {
                        maxSize: 3 * 1024          // by default size is 8 kb, we are making it 3 kb
                    }
                }
            },
            {
                test: /\.txt/,
                type: 'asset/source'
            }
        ]
    }
};
```

