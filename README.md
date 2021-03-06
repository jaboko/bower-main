bower-main
===============

Made to be used with Gulp. Based on asset type, get bower main files as normal file names array and as minimized file names array. If no minified version is found for some files, these file names will be available as a 3rd array so you can minify them yourself. The order of the files is as set in **bower.json**.

It uses [main-bower-files](https://www.npmjs.com/package/main-bower-files), manipulates the result and checks for the availability of a minimized version (in the bower package).

## Installation

```shell
  npm install --save-dev bower-main
```

## Usage

Require the module and get a set of asset files by giving two paramenters: First paramenter is the non-mimified file extension,
like 'js' or 'css'. Second parameter (optional) is the minified file extension, like 'min.js' or 'min.css'.
Here is a usage with JavaScript files:

```js
var bowerMain = require('bower-main');
var bowerMainJavaScriptFilesObject = bowerMain('js','min.js');

var normalJavaScriptFileNamesArray           = bowerMainJavaScriptFilesObject.normal;
var minifiedJavaScriptFileNamesArray         = bowerMainJavaScriptFilesObject.minified;
var minifiedJavaScriptFileNamesNotFoundArray = bowerMainJavaScriptFilesObject.minifiedNotFound;
```

## Minify non-minified scripts in Gulp

Again, this example uses JavaScript files.

```js
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var merge2 = require('merge2');
var bowerMain = require('bower-main');

var bowerMainJavaScriptFiles = bowerMain('js','min.js');

gulp.task('vendorScriptsDevelopment', function() {
  return gulp.src(bowerMainJavaScriptFiles.normal)
    .pipe(concat('vendor-scripts.js'))
    .pipe(gulp.dest('dev'))
});

gulp.task('vendorScriptsProduction', function() {
  return merge2(
    gulp.src(bowerMainJavaScriptFiles.minified),
    gulp.src(bowerMainJavaScriptFiles.minifiedNotFound)
      .pipe(concat('tmp.min.js'))
      .pipe(uglify())
  )
    .pipe(concat('vendor-scripts.min.js'))
    .pipe(gulp.dest('dist'))
});
```

## Copy your Bower dependencies with Gulp

Assuming your Bower dependencies contain various extension files (css, js, fonts, eot, svg, etc.), it may seem heavy to select ALL the possible extensions. So, here is a sample that shows how to...

* Copy the minified JS and CSS scripts when they exists.
* Copy the full JS and CSS scripts when no minified version exist.
* Copy all the other Bower dependencies (no matter their extension).

```js
// Include our plug-ins
var bowerMain = require('bower-main');
var mainBowerFiles = require('main-bower-files');
var exists = require('path-exists').sync;
var gulpIgnore = require('gulp-ignore');

// Create some task
gulp.task( 'copy-bower-dep', function() {

	// Copy minified resources (Bower)
	gulp.src( bowerMain( 'js', 'min.js' ).minified, {base: './my-bower-components/dir'})
			.pipe( gulp.dest( './target/dist/lib' ));
	
	gulp.src( bowerMain( 'css', 'min.css' ).minified, {base: './my-bower-components/dir'})
			.pipe( gulp.dest( './target/dist/lib' ));
	
	// Copy non-minified resources (Bower)
	// Notice we filter these resources to distinguish which one are minified.
	gulp.src( mainBowerFiles(), {base: './my-bower-components/dir'})
			.pipe( gulpIgnore.include( keepNonMinified ))
			.pipe( gulp.dest( './target/dist/lib' ));
});

/*
 * A function that checks whether a Bower file has a minified version.
 */
function keepNonMinified( file ) {
	
	var keep = true;
	if( file.path.match( '\.js$' )) {
		var minPath = file.path.replace( '.js', '.min.js' );
		keep = ! exists( minPath );
		
	} else if( file.path.match( '\.css$' )) {
		var minPath = file.path.replace( '.css', '.min.css' );
		keep = ! exists( minPath );
	}
	
	// gutil.log( file.path + ' => ' + keep );
	return keep;
}
```

Based on [this gist](https://gist.github.com/vincent-zurczak/0ec946faa3dd409c6cfb).

## Issues

If you find a bug, have a feature request or similar, then create an issue on [https://github.com/frodefi/bower-main/issues](https://github.com/frodefi/bower-main/issues).

## LICENSE

MIT © [Frode Fikke](https://github.com/frodefi)
