OVERVIEW
--------
Webpack allows us to use CommonJS syntax as well
as allowing us to code in ES6 by transpiling it to ES5.
The big advantage of CommonJS is the `require()` syntax
which makes it easier to organize our code.

You can use Webpack by itself or combine it with
Gulp so that you can work on Django projects.

#### Using gulp + webpack
```javascript
/************* gulp.js ********************/
// gulp task
gulp.task('wbpk', function() {
  var debug = PATHS.webpack.debug;
  return gulp.src(PATHS.webpack.source_file)
    .pipe(webpack_stream( require('./webpack.config.js') ))
    .pipe(gulp.dest(PATHS.webpack.output_directory));
});

// runserver static file server and watch files
gulp.task('default', ['runserver','css','wbpk'], function () {
    gulp.watch(['../web/frontend/**/*.html'], ['wbpk',reload]);
    gulp.watch('../web/frontend/**/*.css',['css',reload]);
    gulp.watch('../web/frontend/apps/**/*.js',['wbpk',reload]);
});

/****************** webpack.config.js ************************/
// for the configuration reference: https://webpack.github.io/docs/configuration.html
var debug = process.env.NODE_ENV !== "production";

var webpack = require('webpack');
var path = require('path');

var PATHS = {
  webpack: {
    debug: true,
    source_file: '../web/frontend/apps/home/app.js',
    output_filename: 'home.bundle.js',
    output_directory: '../web/frontend/dist/home/js',
    node_modules_directory: __dirname + '/node_modules'
  }
}

var exports = {
      //resolveLoader: { root: path.join(__dirname, "node_modules") },
      resolveLoader: { root: path.resolve(PATHS.webpack.node_modules_directory) },
      module: {
        loaders: [
          { test: /\.html$/, loader: "raw" },
        ]
      },
      entry: {
        app: PATHS.webpack.source_file,
      },
      output: {
        filename: PATHS.webpack.output_filename,
      },
      resolve: { 
        modulesDirectories: [
          path.resolve(PATHS.webpack.node_modules_directory)
          ],
        extensions: ['','.js','.html']
        
      },
      plugins: debug ? [] : [
        new webpack.optimize.DedupePlugin(),
        new webpack.optimize.OccurenceOrderPlugin(),
        // minify and remove comments
        new webpack.optimize.UglifyJsPlugin({ mangle: false, sourcemap: false }),
      ],     
};

module.exports = exports;

```