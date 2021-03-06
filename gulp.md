OVERVIEW
-------
Gulp is a task runner that makes it easier to work on 
frontend code. For example, you can run tasks to watch files and refresh the

If you use Gulp with Browsersync, you can work
on Django templates and have autoreloading in the browser.

INSTALLATION
-----------
```
npm instal -y -g gulp # install globally 
npm install -y -S gulp # install locally in the node_modules directory
```

TASKS
-----
Tasks allow us to break down the transformations
we need for our code such as transpiling, minifying,
and exporting/importing code requirements.

Here's a sample task for running a static server:
```javascript
//Activate virtualenv and run Django server
gulp.task('runserver', function() {
    browserSync.init({
      port: 8080,
      reloadDelay: 300,
      reloadDebounce: 500,
      server: {
            baseDir: "../web/frontend/apps/home",
            routes: {
              "/static":"../web/frontend/dist"
            }
        }
    });
});

// Watch Files For Changes & Reload, the default task
gulp.task('default', ['runserver'], function () {
  gulp.watch(['app/**/*.html'], reload);
});
```

ROUTES (for browsersync)
-------------------------
Routes are url aliases to paths for static files.
You define routes in the `server` object in browsersync
like this:
```javascript
server: {
    routes: {
        "/static":"../web/frontend/dist"
    }
}
// When you goto url /static you can access files in the ../web/frontend/dist directory
```


COMPREHENSIVE EXAMPLE
--------------------
```javascript
/Based on gulpfile.js from Google Web Starter Kit.
//https://github.com/google/web-starter-kit
'use strict';
 
// Include Gulp & Tools We'll Use
var gulp = require('gulp');
var $ = require('gulp-load-plugins')();
var del = require('del');
var runSequence = require('run-sequence');
var browserSync = require('browser-sync');
var pagespeed = require('psi');
var reload = browserSync.reload;
var exec = require('child_process').exec;
var notifier = require('node-notifier');
 
var AUTOPREFIXER_BROWSERS = [
  'ie >= 10',
  'ie_mob >= 10',
  'ff >= 30',
  'chrome >= 34',
  'safari >= 7',
  'opera >= 23',
  'ios >= 7',
  'android >= 4.4',
  'bb >= 10'
];
 
function getRelativePath(absPath) {
    absPath = absPath.replace(/\\/g, '/');
    var curDir = __dirname.replace(/\\/g, '/');
    return absPath.replace(curDir, '');
}
 
function logUglifyError(error) {
    this.emit('end');
    var file = getRelativePath(error.fileName);
    $.util.log($.util.colors.bgRed('Uglify Error:'))
    $.util.log($.util.colors.bgMagenta('file: ') + $.util.colors.inverse(file));
    $.util.log($.util.colors.bgMagenta('line: '+error.lineNumber));
    //remove path from error message
    var message = error.message.substr(error.message.indexOf(' ')+1);
    $.util.log($.util.colors.bgRed(message));
    notifier.notify({ title: 'Gulp message', message: 'Uglify error!' });
}
 
function logCoffeeError(error) {
    this.emit('end');
     var file = getRelativePath(error.filename);
    $.util.log($.util.colors.bgRed('Coffee Error:'))
    $.util.log($.util.colors.bgMagenta('file: ') + $.util.colors.inverse(file));
    $.util.log($.util.colors.bgMagenta('line: '+error.location.first_line+', column: '+error.location.first_column));
    $.util.log($.util.colors.bgRed(error.name+': '+error.message));
    $.util.log($.util.colors.bgMagenta('near: ') + $.util.colors.inverse(error.code));
    notifier.notify({ title: 'Gulp message', message: 'Coffee error!' });
};
 
function logSASSError(error) {
    var file = getRelativePath(error.file);
    $.util.log($.util.colors.bgRed('Sass Error:'))
    $.util.log($.util.colors.bgMagenta('file: ') + $.util.colors.inverse(file));
    $.util.log($.util.colors.bgMagenta('line: '+error.line+', column: '+error.column));
    $.util.log($.util.colors.bgRed(error.message));
    notifier.notify({ title: 'Gulp message', message: 'SASS Error!' });
}
 
// Lint JavaScript
gulp.task('jshint', function () {
  return gulp.src('app/static/scripts/**/*.js')
    .pipe(reload({stream: true, once: true}))
    .pipe($.jshint())
    .pipe($.jshint.reporter('jshint-stylish'));
});
 
//Lint CoffeeScript
gulp.task('coffeelint', function() {
    return gulp.src('app/static/scripts/**/*.coffee')
      .pipe(reload({stream: true, once: true}))
      .pipe($.coffeelint())
      .pipe($.coffeelint.reporter())
})
 
// Optimize Images
gulp.task('images', function () {
  return gulp.src('app/static/images/**/*')
    .pipe($.cache($.imagemin({
      progressive: true,
      interlaced: true
    })))
    .pipe(gulp.dest('dist/static/images'))
    .pipe($.size({title: 'images'}));
});
 
 
// Copy All Files At The Root Level (app)
gulp.task('copy', function () {
  return gulp.src([
    'app/**',
    '!app/**/__pycache__{,/**}',
    '!app/templates{,/**/*.html}',
    '!app/static{/fonts/**,/images/**,/scripts/**,/styles/**}'
  ], {
    dot: true
  }).pipe(gulp.dest('dist'))
    .pipe($.size({title: 'copy'}));
});
 
// Copy Web Fonts To Dist
gulp.task('fonts', function () {
  return gulp.src(['app/static/fonts/**'])
    .pipe(gulp.dest('dist/static/fonts'))
    .pipe($.size({title: 'fonts'}));
});
 
// Compile and Automatically Prefix Stylesheets
gulp.task('styles', function () {
    // For best performance, don't add Sass partials to `gulp.src`
    return gulp.src([
      'app/static/styles/*.scss',
      'app/static/styles/**/*.css',
      'app/static/styles/components/components.scss'
    ])
    .pipe($.sourcemaps.init())
    //.pipe($.changed('.tmp/styles', {extension: '.css'}))
    .pipe($.sass({
      precision: 10,
      onError: logSASSError
    }))
    .pipe($.autoprefixer({browsers: AUTOPREFIXER_BROWSERS}))
    .pipe($.sourcemaps.write())
    .pipe(gulp.dest('.tmp/styles'))
    // Concatenate And Minify Styles
    .pipe($.if('*.css', $.csso()))
    .pipe($.rename({suffix: '.min'}))
    .pipe(gulp.dest('dist/static/styles'))
    .pipe($.size({title: 'styles'}));
});
 
 
//Concat and minify scripts
gulp.task('scripts', function() {
    return gulp.src([
          'app/static/scripts/**/*.coffee',
          'app/static/scripts/**/*.js'
    ])
    .pipe($.sourcemaps.init())
    .pipe($.changed('.tmp/scripts', {extension: '.js'}))
    .pipe($.if('*.coffee', $.coffee({ bare: true })))
      .on('error',  logCoffeeError)
    .pipe($.concat('app.js'))
    .pipe($.sourcemaps.write())
    .pipe(gulp.dest('.tmp/scripts'))
    .pipe($.uglify())
      .on('error', logUglifyError)
    .pipe($.rename({suffix: '.min'}))
    .pipe(gulp.dest('dist/static/scripts'))
    .pipe($.size({title: 'scripts'}));
})
 
 
// Scan Your HTML For Assets & Optimize Them
gulp.task('html', function () {
  var assets = $.useref.assets({searchPath: '{.tmp,app}'});
 
  return gulp.src('app/templates/**/*.html')
    .pipe(assets)
    // Concatenate And Minify JavaScript
    .pipe($.if('*.js', $.uglify({preserveComments: 'some'})))
    // Remove Any Unused CSS
    // Note: If not using the Style Guide, you can delete it from
    // the next line to only include styles your project uses.
    .pipe($.if('*.css', $.uncss({
      html: [
        'app/templates/**/*.html'
      ],
      // CSS Selectors for UnCSS to ignore
      ignore: [
        /.navdrawer-container.open/,
        /.app-bar.open/
      ]
    })))
    // Concatenate And Minify Styles
    // In case you are still using useref build blocks
    .pipe($.if('*.css', $.csso()))
    .pipe(assets.restore())
    .pipe($.useref())
    // Update Production Style Guide Paths
    .pipe($.replace('components/components.css', 'components/main.min.css'))
    // Minify Any HTML
    .pipe($.if('*.html', $.minifyHtml()))
    // Output Files
    .pipe(gulp.dest('dist/templates'))
    .pipe($.size({title: 'html'}));
});
 
// Clean Output Directory
gulp.task('clean', del.bind(null, ['.tmp', 'dist/*', '!dist/.git'], {dot: true}));
 
//Activate virtualenv and run Django server
gulp.task('runserver', function() {
    var isWin = /^win/.test(process.platform);
     var cmd =  '. ../bin/activate';
 
    if (isWin) { //for Windows
        cmd = '..\\Scripts\\activate';
    }
 
    var proc = exec(cmd+' && python app/manage.py runserver');
});
 
gulp.task('runserver:dist', function() {
    var isWin = /^win/.test(process.platform);
     var cmd =  '. ../bin/activate';
 
    if (isWin) { //for Windows
        cmd = '..\\Scripts\\activate';
    }
 
    var proc = exec(cmd+' && python dist/manage.py runserver');
});
 
 
// Build and serve the output from the dist build
gulp.task('serve:dist', ['build', 'runserver:dist'], function () {
  browserSync({
    notify: false,
    proxy: "127.0.0.1:8000"
  });
});
 
 
// Build Production Files
gulp.task('build', ['clean'], function (cb) {
  runSequence('styles', ['jshint', 'coffeelint', 'scripts', 'html', 'images', 'fonts', 'copy'], cb);
});
 
 
// Watch Files For Changes & Reload, the default task
gulp.task('default', ['styles', 'jshint', 'coffeelint', 'scripts', 'runserver'], function () {
  browserSync({
    notify: false,
    proxy: "127.0.0.1:8000"
  });
 
  gulp.watch(['app/**/*.html'], reload);
  gulp.watch(['app/static/styles/**/*.{scss,css}'], ['styles', reload]);
  gulp.watch(['app/static/scripts/**/*.js'], ['jshint']);
  gulp.watch(['app/static/scripts/**/*.coffee'], ['coffeelint']);
  gulp.watch(['app/static/scripts/**/*.{js,coffee}'], ['scripts', reload]);
  gulp.watch(['app/static/images/**/*'], reload);
});
 
 
// Run PageSpeed Insights
gulp.task('pagespeed', function (cb) {
  // Update the below URL to the public URL of your site
  pagespeed.output('example.com', {
    strategy: 'mobile',
    // By default we use the PageSpeed Insights free (no API key) tier.
    // Use a Google Developer API key if you have one: http://goo.gl/RkN0vE
    // key: 'YOUR_API_KEY'
  }, cb);
});
```