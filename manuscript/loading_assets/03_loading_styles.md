# Loading Styles

Loading styles is a standard operation. There are a lot of variants depending on the styling approach you use, though. I'll cover the most common options next. You can combine these approaches with the `ExtractTextPlugin` to get better output for your production build.

## Loading CSS

Loading vanilla CSS is fairly straightforward as you can see in the example below. It parses the styles referred by the project while making sure only files ending with `.css` are matched by the loaders. The definition then applies both *style-loader* and *css-loader* on it:

**webpack.config.js**

```javascript
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader']
}
```

When webpack evaluates the files, first [css-loader](https://www.npmjs.com/package/css-loader) goes through possible `@import` and `url()` statements within the matched files and treats them as regular `require`. This allows us to rely on various other loaders, such as [file-loader](https://www.npmjs.com/package/file-loader) or [url-loader](https://www.npmjs.com/package/url-loader). We will see how these work in the next chapters.

*file-loader* generates files, whereas *url-loader* can create inline data URLs for small resources. This can be useful for optimizing application loading. You avoid unnecessary requests while providing a slightly bigger payload. Small improvements can yield large benefits if you depend on a lot of small resources in your style definitions.

After *css-loader* has done its part, *style-loader* picks up the output and injects the CSS into the resulting bundle. This will be inlined JavaScript by default. This is something you want to avoid in production usage. It makes sense to use `ExtractTextPlugin` to generate a separate CSS file in this case as we saw earlier.

Setting up other formats than vanilla CSS is simple as well. I'll discuss specific examples next.

T> If you want to enable sourcemaps for CSS, you should enable `sourceMap` option for *css-loader* and set `output.publicPath` to an absolute url. In case you have more loaders, each needs to enable sourcemap support separately! *css-loader* [issue 29](https://github.com/webpack/css-loader/issues/29) discusses this problem further.

## Loading LESS

![Less](images/less.png)

[Less](http://lesscss.org/) is a popular CSS processor that is packed with functionality. In webpack using Less doesn't take a lot of effort. [less-loader](https://www.npmjs.com/package/less-loader) deals with the heavy lifting. You should install [less](https://www.npmjs.com/package/less) as well given it's a peer dependency of *less-loader*. Consider the following minimal setup:

```javascript
{
  test: /\.less$/,
  use: ['style-loader', 'css-loader', 'less-loader']
}
```

There is also support for Less plugins, sourcemaps, and so on. To understand how those work you should check out the project itself.

## Loading SASS

![Sass](images/sass.png)

[Sass](http://sass-lang.com/) is a popular alternative to Less. You should use [sass-loader](https://www.npmjs.com/package/sass-loader) with it. Remember to install [node-sass](https://www.npmjs.com/package/node-sass) to your project as the loader has a peer dependency on that. webpack doesn't take much configuration:

**webpack.config.js**

```javascript
{
  test: /\.scss$/,
  use: ['style-loader', 'css-loader', 'sass-loader']
}
```

Sometimes you might see imports like `@import "~bootstrap/css/bootstrap";` in SASS code. The tilde (`~`) tells webpack that it's not a relative import as by default in SASS. If tilde is included, it will perform a lookup against `node_modules` (default setting) although this is configurable through the [resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules) field.

Check out the loader for more advanced usage.

T> If you want more performance especially during development, check out [fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader).

### Imports in LESS and SASS

If you import one LESS/SASS file from an other, use the exact same pattern as anywhere else. webpack will dig into these files and figure out the dependencies.

```less
@import "./variables.less";
```

You can also load LESS files directly from your node_modules directory. This is handy with libraries like Twitter Bootstrap:

```less
$import "~bootstrap/less/bootstrap";
```

## Loading Stylus and YETICSS

![Stylus](images/stylus.png)

Stylus is yet another example of a CSS processor. It works well through [stylus-loader](https://github.com/shama/stylus-loader). There's also a pattern library known as [yeticss](https://www.npmjs.com/package/yeticss) that works well with it. Consider the following configuration:

**webpack.config.js**

```javascript
const common = {
  ...
  module: {
    rules: [
      {
        test: /\.styl$/,
        use: ['style-loader', 'css-loader', 'stylus-loader']
      }
    ]
  },
  plugins: [
    new webpack.LoaderOptionsPlugin({
      options: {
        // yeticss
        stylus: {
          use: [require('yeticss')]
        }
      }
    })
  ]
};
```

To start using yeticss with Stylus, you must import it to one of your app's .styl files:

```javascript
@import 'yeticss'

//or
@import 'yeticss/components/type'
```

## PostCSS

![PostCSS](images/postcss.png)

[PostCSS](https://github.com/postcss/postcss) allows you to perform transformations over CSS through JavaScript plugins. You can even find plugins that provide you Sass-like features. PostCSS can be thought as the equivalent of Babel for styling. It can be used through [postcss-loader](https://www.npmjs.com/package/postcss-loader) with webpack.

The example below illustrates how to set up autoprefixing using it. It also sets up [precss](https://www.npmjs.com/package/precss), a PostCSS plugin that allows you to use Sass-like markup in your CSS. You can mix this technique with other loaders to allow autoprefixing there.

**webpack.config.js**

```javascript
const autoprefixer = require('autoprefixer');
const precss = require('precss');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader']
      }
    ]
  }
};
```

In addition to webpack configuration, we need to set up some for PostCSS:

**postcss.config.js**

```javascript
module.exports = {
  plugins: {
    autoprefixer: {},
    precss: {}
  }
};
```

T> For this to work, you will have to remember to include [autoprefixer](https://www.npmjs.com/package/autoprefixer) and [precss](https://www.npmjs.com/package/precss) to your project.

### cssnext

![cssnext](images/cssnext.jpg)

[cssnext](https://cssnext.github.io/) is a PostCSS plugin that allows us to experience the future now. There are some restrictions, but it may be worth a go. In webpack it is simply a matter of installing [cssnext-loader](https://www.npmjs.com/package/cssnext-loader) and attaching it to your CSS configuration. In our case, you would end up with the following:

**webpack.config.js**

```javascript
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader', 'cssnext-loader']
}
```

Alternatively, you could consume it through *postcss-loader* as a plugin if you need more control.

## Conclusion

Loading style files through webpack is fairly straight-forward. It supports even advanced techniques like [CSS Modules](https://github.com/css-modules/webpack-demo). CSS Modules make CSS local by default. This can be a great boon especially for developers who work with component oriented libraries. The approach works beautifully there.
