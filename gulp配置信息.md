###配置gulp
####先安装好各个依赖的插件
    1.首先安装node环境，安装gulp插件使用npm进行
    2.进入工作目录，在此处打开命令行
    3.使用npm管理该文件夹，npm init -y 快速生成package.json文件
    4.通过npm安装各个gulp的插件
    npm install gulp --save-dev //基础库
    npm install gulp-htmlmin --save-dev  //html压缩
    npm install gulp-csso --save-dev  //css压缩
    npm install jshint gulp-jshint --save-dev  //js检查
    npm install gulp-uglify --save-dev  //js压缩
    npm install gulp-concat --save-dev  //文件合并
    npm install gulp-clean --save-dev  //清空文件夹
    npm install gulp-imagemin --save-dev  //图片压缩
    npm install gulp-rename --save-dev  //文件重命名
    npm install gulp-rev --save-dev  //更改版本名
    npm install gulp-rev-collector --save-dev
    //gulp-rev的插件，用于HTML模板更改引用路径
####在当前文件夹的根目录下建立gulpfile.js文件，进行配置
    1.首先将各个依赖的插件引入
    var gulp = require('gulp'),  //基础库
        htmlmin = require('gulp-htmlmin'),//html压缩
        csso = require('gulp-csso'), //css压缩
        jshint = require('gulp-jshint'), //js检查
        uglify = require('gulp-uglify'), //js压缩
        concat = require('gulp-concat'), //文件合并
        clean = require('gulp-clean'), //清空文件夹
        imagemin = require('gulp-imagemin'), //图片压缩
        rename = require('gulp-rename'), //文件重命名
        rev = require('gulp-rev'), //更改版本名
        revCollector = require('gulp-rev-collector'); //gulp-rev的插件，用于HTML模板更改引用路径
    2.开始配置任务
    // 1.html压缩   htmlmin使用
        gulp.task('html', function () {  //task方法是创建任务的实现方式
            var options = {
                collapseWhitespace:true,
                collapseBooleanAttributes:true,
                removeComments:true,
                removeEmptyAttributes:true,
                removeScriptTypeAttributes:true,
                removeStyleLinkTypeAttributes:true,
                minifyJS:true,
                minifyCSS:true
            };
            return gulp.src('index.html')
            //src方法获取文件，此处获取同文件夹下的index，html文件
                .pipe(htmlmin(options)) //去除html中的空白符
                .pipe(gulp.dest('dist/'));
                //dest方法，指定压缩后的文件存放路径，此时：如果没有dist文件夹会自动创建
        });
        //2.css压缩
        gulp.task('styles', function () {
            return gulp.src('./css/*.css')  //选择所有css目录下的css文件
                .pipe(csso()) //  压缩css代码
                // 此处不要修改文件名称，不然下面rev任务自动替换引用文件时会找不到要替换的内容
                // .pipe(rename(function (path) {
                //     path.basename += '.min';
                //     path.extname = '.css'
                // }))
                .pipe(rev())
                .pipe(gulp.dest('./dist/css'))//  压缩后保存目录
                .pipe(rev.manifest())
                .pipe(gulp.dest('./rev/css'));
        });
        // 3.js语法检测、代码合并、压缩
        gulp.task('scripts', function () {
            //合并操作的方法，此处不执行
            //gulp.src(['./demo.js','./index.js'])//要合并操作的文件
            //.pipe(concat('all.js'))  //合并后名称
            return gulp.src('./js/*.js')
                .pipe(jshint()) // 进行检查
                .pipe(jshint.reporter('default')) // 对代码进行报错提示
                .pipe(uglify())   //js代码压缩
                .pipe(rev())
                .pipe(gulp.dest('./dist/js'))//  压缩后保存目录
                .pipe(rev.manifest())
                .pipe(gulp.dest('./rev/js'));
        });
        // 4.图片:压缩
        gulp.task('images', function() {
            gulp.src('./images/*.*')  //选的images文件夹下的所有图片
                .pipe(imagemin())
                .pipe(gulp.dest('dist/images'))
        });
        //5. rev使用，自动改变HTML文件中的引用路径
        gulp.task('rev',function () {
            return gulp.src(['./rev/**/*.json','./dist/index.html'])
                .pipe(revCollector({
                    replaceReved:true
                }))
                .pipe(rename(function (path) {
                    path.basename += '.min';
                    path.extname = '.html'
                }))
                .pipe(gulp.dest('./dist'));
        });
        // 主任务，只需在命令行执行此任务就能实现压缩代码，rev任务除外
        gulp.task('default', ['html', 'styles', 'scripts','images', 'rev'], function () {
            //在执行default任务时，先执行['html', 'styles', 'scripts', 'images']
            //的任务，然后再执行下面的代码(称之为任务依赖)
            //当指定的文件发送改变时执行指定任务
            gulp.watch('./index.html', ['html']);
            gulp.watch('./css/*.css', ['styles']);
            gulp.watch('./js/*.js', ['scripts']);
        });
###注意：在执行完default主任务后没有实现HTML中路径自动改变，需要单独执行rev任务才能为HTML引入的文件自动加上版本号
