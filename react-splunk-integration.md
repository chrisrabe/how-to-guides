# React Splunk Integration

## Overview
Instructions for integrating React with Splunk. Exact versioning will be used when installing packages to future-proof the tutorial.

## Prerequisites
- Installed Splunk
- Installed NodeJS and NPM
- Splunk application is created

# Instructions

## Step 1: Create your homepage (optional)

*Note 1: Skip this step if you already have a dashboard* 

*Note 2: This step assumes you're building from a bare Splunk app*

1. Open `$SPLUNK_HOME/etc/apps/<your-app>` in your IDE
2. Modify `default/data/ui/nav` to

```xml
<nav search_view="search">
  <view name="homepage" default='true' />
</nav>
```

3. Create a `homepage.xml` inside `default/data/ui/views`
4. Paste the following as the contents of your `homepage.xml`

```xml
<dashboard script="dashboard_file.js">
  <label>Homepage</label>
  <row>
    <panel>
      <html><div id="root"></div></html>
    </panel>
  </row>
</dashboard>
```

5. Restart your Splunk (see Appendix A)

## Step 2: Setup React UI files and directory

1. Create your `appserver/static` directory from your splunk application directory
```
mkdir -p appserver/static
```
2. Go to your `appserver/static` folder
3. Create your react project folder (in this tutorial, we'll call it `react-ui`)
4. Go to your folder by typing `cd react-ui`
5. Type `npm init` (this makes `react-ui` a node app)
6. Install and setup `babel` (see section 2.1)
7. Install and setup `webpack` (see section 2.2)
8. Install and setup `React` (see section 2.3)
9. Setup `dashboard_file.js` (see section 2.4)

### 2.1 Install and setup Babel
1. Install the required dependencies by typing
```
npm install --save-dev @babel/cli@7.1.0 @babel/core@7.1.0 @babel/preset-env@7.1.0 @babel/preset-react@7.0.0
```
2. Create a file called `.babelrc` in the root folder of your node app
3. Enter the following contents
```json
{
    "presets": ["@babel/env", "@babel/preset-react"]
}
```

### 2.2 Install and setup Webpack
1. Install the required dependencies by typing
```
npm install --save-dev webpack@4.19.1 webpack-cli@3.1.1 html-webpack=plugin@3.2.0 babel-loader@8.0.2 css-loader@1.0.0 html-loader@0.5.5 style-loader@0.23.0
```
2. Create a file called `webpack.config.js`
3. Enter the following contents into the file
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/App.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
        library: 'App',
        libraryTarget: 'umd'
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                loader: 'babel-loader'
            },
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader']
            },
            {
                test: /\.html$/,
                use: [
                    {
                        loader: 'html-loader'
                    }
                ]
            }
        ]
    },
    devtool: 'cheap-module-eval-source-map',
    devServer: {
        contentBase: path.join(__dirname, 'dist')
    },
    plugins: [
        new HtmlWebpackPlugin()
    ]
};
```
4. Inside your `package.json`, modify the `scripts` property to the following
```json
    ...
    "scripts": {
        "build": "webpack",
        "watch": "webpack -d --watch --progress"
    },
    ...
```

The `build` script manually creates `bundle.js` using webpack. The `watch` script automatically updates `bundle.js` if there are any code changes.

### 2.3 Install and setup react
1. Install React and React DOM
```
npm install --save react@16.5.2 react-dom@16.5.2
```
2. Create a `src` folder
3. Create an `App.js` file inside the `src` folder
4. Enter the following contents in your `App.js` file
```js
import React from 'react';

class App extends React.Component {
    render() {
        return (
            <div>
                <p>Hello world!</p>
            </div>
        )
    }
}

export default App;
```
5. While in your `react-ui` directory, run `npm run build` in your command line

### 2.4 Setup `dashboard_file.js`

1. From your `appserver/static` folder, create `dashboard_file.js`
2. Enter the following code in the file
```js
require.config({
    paths: {
        react: '/static/app/react-splunk-poc/react-ui/node_modules/react/umd/react.production.min',
        'react-dom': '/static/app/react-splunk-poc/react-ui/node_modules/react-dom/umd/react-dom.production.min',
        app: '/static/app/react-splunk-poc/react-ui/dist/bundle'
    }
});

require([
    'underscore',
    'backbone',
    'splunkjs/mvc',
    'jquery',
    'react',
    'react-dom',
    'app'
], function(_, Backbone, mvc, $, React, ReactDOM, App) {
    ReactDOM.render(
        React.createElement(App.default),
        document.getElementById('root')
    );
});
```
3. Open Splunk from your web browser
4. Go to the following URL `https://<splunk-url>/en-GB/_bump`
5. Press the `bump` button
6. Open your Splunk application in your web browser. You should see "Hello World" printed on one of the panels

## Step 3: Disable Splunk Caching

1. Create a file called `web.conf` inside your `default` folder
2. Enter the following settings:
```
[settings]
 js_no_cache = true
 cacheBytesLimit = 0
 cacheEntriesLimit = 0
 max_view_cache_size = 0
 auto_refresh_views = 1
```
3. Restart the Splunk server

## Step 4: Develop your UI
See `Appendix B`

# Explanation

## Step 1 Explanation
In this section, we created a Splunk application with a single dashboard called `homepage`. Inside the homepage, the dashboard references a file called `dashboard_file.js` and contains a single HTML panel with a `root` div. 

The `dashboard_file.js` script is responsible for injecting React elements into the dashboard. The div has an id called `root` so that the script can identify where to insert the React elements.

## Section 2.2 Explanation
In this section, we set up babel for React. This allows us to compile bits of code in the `src` folder to pure Javascript.

## Section 2.3 Explanation
In this section, we created a webpack configuration and created a `build` command. Essentially, the `build` command executes `webpack` and the configuration we created tells webpack to compile everything inside the Javascript folder into a *big* javascript file called `bundle.js` inside the `dist` folder.

## Section 2.4 Explanation
As mentioned above, the `dashboard_file.js` is responsible for injecting React elements into the dashboard. It does this through the `react-dom` library.

## Step 3 Explanation
We have to create the `web.conf` file so that when you make front-end changes, you wouldn't have to hit the `bump` button all the time.

# Appendix

## Appendix A: Restaring Splunk
**Note:** If you can't see the settings bar, go to `https://<SPLUNK_URL>/en-GB/manager/<APP>/control`

1. Open Splunk via your Web Browser
2. Log in
3. Go to `Settings > Server Controls`
4. Press `Restart Splunk`

## Appendix B: Developing the UI
When developing your React front-end, you need to have two things running:
1. The `watch` script
2. Browser developer console

You can run the `watch` script by typing `npm run watch` from your `react-ui` directory.

When you have the browser developer console open, ensure that you have **caching disabled**. For Chrome, this can be found in the `Network` tab. **You need to have the developer console open at all times, otherwise you wouldn't be able to see changes.**

![disable-cache](images/react-splunk-disable-cache.png)