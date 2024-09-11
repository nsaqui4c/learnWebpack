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
        publicPath: 'dist/'                       // path to look for files imported by bundles
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

### Loaders
* We use loaders to tell webpack how to use provided files.
* There are no in built loaders, we need to npm install loaders to utillize them
***npm i -D style-loader css-loader sass-loader***
BELOW CODE WILL ADD SCSS styles TO HEAD SECTION In HTML
```js
//indesx.js
import HelloWorldButton from './components/hello-world-button/hello-world-button.js';

const helloWorldButton = new HelloWorldButton();
helloWorldButton.render();

/////////////////////  hello-world-button.js  //////////////////////////////////////////////////
import './hello-world-button.scss';

class HelloWorldButton {
    render() {
        const button = document.createElement('button');
        const body = document.querySelector('body');
        button.innerHTML = 'Hello world';
        button.onclick = function () {
            const p = document.createElement('p');
            p.innerHTML = 'Hello world';
            p.classList.add('hello-world-text');
            body.appendChild(p);
        }
        button.classList.add('hello-world-button');
        body.appendChild(button);
    }
}

export default HelloWorldButton;
```
```js
$font-size: 20px;
$button-background-color: green;
$button-font-color: white;
$text-font-color: red;

.hello-world-button {
    font-size: $font-size;
    padding: 7px 15px;
    background: $button-background-color;
    color: $button-font-color;
    outline: none;
}

.hello-world-text {
    color: $text-font-color;
    font-weight: bold;
}
```
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
                parser: {
                    dataUrlCondition: {
                        maxSize: 3 * 1024
                    }
                }
            },
            {
                test: /\.txt/,
                type: 'asset/source'
            },
            {
                test: /\.css$/,
                use: [
                    'style-loader', 'css-loader'
                ]
            },
            {
                test: /\.scss$/,
                use: [
                    'style-loader', 'css-loader', 'sass-loader'
                ]
            }
        ]
    }
};
```
 
### Babel
* It is third party library that is mainly used to convert ECMAScript 2015+ code into a backwards compatible version of JavaScript in current and older browsers or environments.
* Transform syntax
* Polyfill features that are missing in your target environment (through a third-party polyfill such as core-js)
* Source code transformations (codemods)
* Babel can convert JSX syntax  (add @babel/preset-react to your Babel configuration.)
* npm install @babel/core babel-loader @babel/preset-env @babel/plugin-proposal-class-properties --save-dev
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
                parser: {
                    dataUrlCondition: {
                        maxSize: 3 * 1024
                    }
                }
            },
            {
                test: /\.txt/,
                type: 'asset/source'
            },
            {
                test: /\.css$/,
                use: [
                    'style-loader', 'css-loader'
                ]
            },
            {
                test: /\.scss$/,
                use: [
                    'style-loader', 'css-loader', 'sass-loader'   // loader load from right to left-> first saas, then css then style
                ]
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [ '@babel/env' ],
                        plugins: [ '@babel/plugin-proposal-class-properties' ]     // using this to enable using class variable. not all
                                                                                  // browser support class variable. they support class method only
                    }
                }
            }
        ]
    }
};
```
### Plugins
* plugins are additional js lib that do everything that loaders can do
* They can also modify how the bundles themsleves are created.
  * eg.  uglifyJSPlugin take the bundle.js and minimize the content to decrease size.
* We can add plugin to create seperate file for sass and css called -> MiniCssExtractPlugin  (we need to change style-loader in rule also)


```js
const path = require('path');
const TerserPlugin = require('terser-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.[contenthash].js',
        path: path.resolve(__dirname, './dist'),
        publicPath: 'dist/'
    },
    mode: 'none',
    module: {
        rules: [
            {
                test: /\.(png|jpg)$/,
                type: 'asset',
                parser: {
                    dataUrlCondition: {
                        maxSize: 3 * 1024
                    }
                }
            },
            {
                test: /\.txt/,
                type: 'asset/source'
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader, 'css-loader'    //change loader also according to plugin
                ]
            },
            {
                test: /\.scss$/,
                use: [
                    MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader'
                ]
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [ '@babel/env' ],
                        plugins: [ '@babel/plugin-proposal-class-properties' ]
                    }
                }
            }
        ]
    },
    plugins: [
        new TerserPlugin(),
        new MiniCssExtractPlugin({
            filename: 'styles.[contenthash].css'
        }),
        new CleanWebpackPlugin({                             // new CleanWebpackPlugin() -> for default setting
            cleanOnceBeforeBuildPatterns: [                  // if no option passsed it will clean output folder only
                '**/*',                                      // configuring to clean output folder
                path.join(process.cwd(), 'build/**/*')       // This will clean build folder also
            ]
        })
    ]
};
```
## webpack dev server
* We can add server to host the build project in given port.
```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist'),
        publicPath: ''
    },
    mode: 'development',
    devServer: {                                                // Server configuration
        port: 9000,
        static: {
            directory: path.resolve(__dirname, './dist'),
        },
        devMiddleware: {
            index: 'index.html',                               // Entr point
            writeToDisk: true                                  // write the output file also in the output folder. Otherwise it will directly host the output.
        }
    },
    module: {
        rules: [
            {
                test: /\.(png|jpg)$/,
                type: 'asset',
                parser: {
                    dataUrlCondition: {
                        maxSize: 3 * 1024
                    }
                }
            },
            {
                test: /\.txt/,
                type: 'asset/source'
            },
            {
                test: /\.css$/,
                use: [
                    'style-loader', 'css-loader'
                ]
            },
            {
                test: /\.scss$/,
                use: [
                    'style-loader', 'css-loader', 'sass-loader'
                ]
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [ '@babel/env' ],
                        plugins: [ '@babel/plugin-proposal-class-properties' ]
                    }
                }
            },
            {
                test: /\.hbs$/,
                use: [
                    'handlebars-loader'
                ]
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Hello world',
            template: 'src/index.hbs',
            description: 'some description'
        })
    ]
};
```
## Extra
* We can add hash to each file name for creating new file-name in case of any change file
  * for this we just need to add [contenthash] in file name ->
```js
 output: {
        filename: 'bundle.[contenthash].js',
        path: path.resolve(__dirname, './dist'),
        publicPath: 'dist/'
    },
```
* We can have multiplle config file. To use them we need to change script in package.json
```js
build : "webpack --config webpack.dev.config.js"
```

* we can add mode to webpack (dev or prod)
  * we can check the current mode in js using ***process.env.NODE_ENV***
