---
title: node笔记整理之fs和path
date: 2018-03-06 14:27:20
categories: node
tags: 
  - fs
  - path
---

## FS模块
### 打开文件
`fs.open(path, flags[, mode], callback)`

```javascript
// flags: 文件打开模式，读取为'r'，写入为'w'，追加为'a'
// 读取
fs.open('./readfile.txt', 'r', (err, fd) => {
    if (err) {
        console.log('发生错误: ' + err);
    } else {
        let buff = Buffer.allocUnsafe(255);
        // fs.read() @params(文件描述，指定读入的缓存，写入缓存开始位置，写入长度，读取文件开始位置,callback)
        // callback() @params(错误对象，实际长度，被读取缓存的对象)
        fs.read(fd, buff, 0, 28, 0, (err, bytesRead, buffer) => {
            let newBuff = buffer.slice(0, bytesRead).toString();
            console.log(bytesRead, newBuff);
        });
    }
});
// 写入
fs.open('./writefile.txt', 'a', (err, fd) => {
    if (err) {
        console.log('发生错误: ' + err);
    } else {
        let buff = Buffer.from('打开后写入：nodeJs博大精深~\r\n');
        // fs.write() @params(文件描述，指定读出的缓存，读取缓存开始位置，读取长度|null默认长度，写入文件开始位置|null上次位置,callback)
        // callback() @params(错误对象，实际长度，被读取缓存的对象)
        fs.write(fd, buff, 0, null, null, (err, written, buffer) => {
            if (err) console.log('写文件失败');
            console.log('写文件成功', written, buffer);
            fs.fsync(fd, err => { if (err) console.log('fsync错误：' + err) });
            fs.close(fd, err => { if (err) console.log('close错误：' + err) });
        });
    }
});

```
<!--more-->
### 读写文件
```javascript
const buff = Buffer.from('nodejs博大精深');
const addStr = Buffer.from('\r\n这是追加的内容@#$%');
let options = {
    flag: 'a+',
    encoding: 'utf8',
    mode: '0666'
};

fs.readFile('./readfile.txt', 'utf8', (err, data) => {
    if (err) {
        console.log('发生错误: ' + err);
    } else {
        console.log('开始读取内容: ' + data);
    }
});

fs.writeFile('./writefile.txt', addStr, options, function(err) {
    if (err) {
        console.log('发生错误: ' + err);
    } else {
        console.log('写文件成功');
    }
});
```

### 文件流方式读文件
`fs.createReadStream(path,[options])`
```javascript
/*
 * 
 * fs.createReadStream(path,[options]) 返回ReadStream对象
 * @params path 指定需要读取完整路径以及文件名
 * @params options [flags,encoding,autoclose,start,end]
 * 
 * ReadStream对象
 * @event readable 当可以从流中读取数据时触发
 * @event open 当文件被打开时触发
 * @event data 当读取到新的数据时触发 参数为存放已读取到的数据缓存区对象或者字符串
 * @event end  当读取完所有数据时触发 此时data事件不再被触发
 * @event error 当读取数据过程中出现错误时触发
 * @event close 当用于读取流数据的对象被关闭时触发
 * @method read 用于读取数据
 * @method setEncoding 指定用何种编码读取数据
 * @method pause 通知对象停止data事件触发
 * @method resume 通知对象恢复触发data事件
 * @method pipe 设置数据通道，取出所有流数据传输给通道另一端对象
 * @method unpipe 取消pipe方法设置的通道
 * @method unshift 取消解析器绑定 流数据采用其他方式解析
 */
let rfile = fs.createReadStream('./writefile.txt', { start: 0, end: 20 });
rfile.pause();
setTimeout(() => {
    rfile.resume();
}, 1000);
rfile.resume();
rfile.on('open', fd => {
    console.log('开始读取文件');
});
rfile.on('data', data => {
    console.log('读取到数据：' + data);
});
rfile.on('end', () => {
    console.log('文件全部读取完毕');
});
rfile.on('close', () => {
    console.log('文件被关闭');
});
rfile.on('error', err => {
    console.log('读取文件失败' + err);
});
```

### 文件流方式写文件  
`fs.createWriteStream(path,[options])`
```javascript
/*
 * 
 * fs.createWriteStream(path,[options]) 返回WriteStream对象
 * @params path 指定需要写入的完整路径以及文件名
 * @params options [flags,encoding,start]
 * 
 * WriteStream对象
 * @event drain 操作系统缓存区中的数据已全部输出到目标对象中，可以继续向前者写入数据
 * @event finish end方法被调用且数据被全部写入操作系统缓存区时候触发
 * @event pipe  读取数据的对象的pipe方法被触发时候
 * @event unpipe 读取数据的对象的unpipe方法被触发时候
 * @event error 当读取数据过程中发生错误
 * @method write(chunk,[encoding],[callback]) 用于写入数据 
 * * * * * @params chunk 要写入的的数据 Buffer或String
 * * * * * @params encoding 指定编码写入
 * * * * * @params callback 写入完毕回调 无参数
 * * * * * @return 表示操作系统缓存是否未满 还可以写入数据
 * @method end 当没有数据再被写入流中调用
 */
let writable = fs.createWriteStream('./writefile.txt', { flags: 'a', encoding: 'utf8' });
let flag = writable.write('create node stream txt\r\n', 'utf8', () => {
    console.log('write写入成功');
});
console.log(flag); //true || false
```

### 图片文件读写
```javascript
fs.readFile('./sample.png', 'base64', (err, data) => {
    if (err) {
        console.log('读文件发生错误: ' + err);
    } else {
        fs.writeFile('./copy.jpg', data.toString(), 'base64', (err) => {
            if (err) {
                console.log('写文件发生错误: ' + err);
            } else {
                console.log('写文件成功');
            }
        });
    }
});
```

### 创建并读取目录
```javascript
/*
 * fs.mkdir(path,[mode],callback) callback @params(错误对象)
 * fs.readdir(path,callback) callback @params(错误对象, 所有文件名)
 */
fs.mkdir('./test/childfile', err => {
    if (err) console.log(err);
    console.log('创建目录成功');
});
fs.readdir('./', (err, files) => {
    if (err) console.log(err);
    console.log(files);
});
```

### 查看文件目录信息
```javascript
/*
 * fs.stat(path,callback) & fs.lstat(path,callback) 
 * callback @params(错误对象, fs.Stats对象)
 * fs.Stats对象 
 * @methods isFile() 是否是文件
 * @methods isDirectory() 是否是目录
 * @methods isSymbolicLink() 是否是符号链接
 * @property mode 数字形式权限表示
 * @property nlink 硬链接数量
 * @property size 文件大小
 * @property atime 访问时间
 * @property mtime 修改时间
 * @property ctime 创建时间
 */
fs.stat('./test', (err, stats) => {
    if (!err) console.log(stats);
});
```

### 检查文件目录是否存在
```javascript
fs.exists('./message', exists => {
    console.log(exists);
});
```

### 获取文件目录绝对路径
```javascript
/*
 * fs.realpath(path,[cache],callback)
 * @params path 要查看文件目录的完整路径
 * @params cache 预先指定的路径
 * callback @params err错误对象
 * callback @params resolvePath文件或者目录的绝对路径
 */
fs.realpath('./test/childfile', (err, resolvePath) => {
    if (err) throw err;
    console.log(resolvePath);
});
```

### 修改文件目录访问修改时间
```javascript
/*
 * fs.utimes(path,atime,mtime,callback)
 * fs.futimes(fd,atime,mtime,callback)
 * @params atime 指定修改后的访问时间
 * @params mtime 指定修改后的修改时间
 * @params fd 使用open方法打开后返回的文件描述
 */
fs.utimes('./test/childfile', new Date(), new Date(), err => {
    if (err) throw err;
    console.log('修改文件时间成功~');
});
```

### 修改文件目录的访问权限
```javascript
/*
 * fs.chmod(path,mode,callback)
 * fs.chmod(fd,mode,callback)
 * @params mode 数字形式的权限表示法
 * @params fd 使用open方法打开后返回的文件描述
 * callback @params err 错误对象
 * 0+ r=4 w=2 x=1 
 * user group other
 */
fs.chmod('./test', 0740, err => {
    if (err) throw err;
    console.log('文件权限修改成功!');
});
```

### 移动文件、目录
```javascript
/*
 * fs.rename(oldpath,newpath,callback)
 * callback @params err 错误对象
 */
fs.rename('./readfile.txt', './test/childfile/newfile.txt', err => {
    if (err) throw new Error('移动失败' + err);
    console.log('移动文件成功');
});
```

###  截断文件、修改文件大小
```javascript
/*
 * fs.truncate(filename,len,callback)
 * fs.truncate(fd,len,callback)
 * @params filename 被截断的完整路径文件名
 * @params len 截断后的文件尺寸（字节）
 * @params fd open方法打开返回的文件描述
 * callback @params err 错误对象
 */
fs.truncate('./truncate.txt', 30, err => {
    if (err) throw err;
    fs.stat('./truncate.txt', (err, stats) => {
        if (err) throw err;
        console.log(stats.size);
    });
});
```

### 删除空目录
```javascript
/*
 *  fs.rmdir(path,callback)
 */
fs.rmdir('./rmdir', err => {
    if (err) throw err;
    console.log('删除目录成功');
});
```

### 监视文件、目录 取消监视
监视 `fs.watchFile(filename,[options],listener)`
监视 `fs.watch(filename,[options],callback)`
取消监视 `fs.unwatchFile(filename,[listener])`

```javascript
/*
 * fs.watchFile(filename,[options],listener)
 * fs.unwatchFile(filename,[listener])
 * @params filename 监听文件的完整目录文件名
 * @params listener 监听的文件发生改变执行的回调，同一文件可监听多个回调
 * @params [listener] 取消监听的文件执行的某一回调
 * options [Object] @property persistent 表明当文件正在被监视时，进程是否应该继续运行
 * options [Object] @property interval 表示目标应该每隔多少毫秒被轮询
 * listener @params curr 修改后的文件fs.Stats对象
 * listener @params prev 修改前的文件fs.Stats对象
 */
fs.watchFile('./readfile.txt', (curr, prev) => {
    if (Date.parse(prev.ctime) === 0) {
        console.log('文件被创建~');
    } else if (Date.parse(curr.ctime) === 0) {
        console.log('文件被删除~');
    } else if (Date.parse(curr.mtime) !== Date.parse(prev.mtime)) {
        console.log('文件被修改');
    }
});

// 监听另一方法
// fs.watch(filename,[options],callback)
// callback(event,filename)
// @params event = 'rename' 移动删除重命名 || 'change'内容改动
// @params filename 指定目录中发生改动的文件完整路径以及文件名

let watcher = fs.watch('./test/', (event, filename) => {
    console.log(event, filename);
});
watcher.close(); //停止监听
```
### 创建删除文件[硬链接] 不同文件名指向同一文件
```javascript
/*
 * fs.link(srcpath,distpath,callback)
 * fs.unlink(path,callback)
 * @params srcpath 创建硬链接源
 * @params distpath 创建硬链接目标
 * callback @params err 错误对象
 */
fs.link('./writefile.txt', './test/childfile/link2.txt', err => {
    if (err) throw err;
    console.log('创建硬链接成功');
});
fs.unlink('./symlink', err => {
    if (err) throw err;
    console.log('删除硬链接成功');
});
```

### 创建查看[符号链接] 创建替身 打开符号链接自动指向源
```javascript
/*
 * fs.symlink(srcpath,distpath,[type],callback)
 * @params type 默认file [,dir]创建类型
 * @params srcpath 创建符号链接源
 * @params distpath 创建符号链接源
 */
fs.symlink('./test', './symlink', err => {
    if (err) throw err;
    console.log('创建符号链接成功');
});
```
### 读取符号链接中包含另一个文件的目录、文件名
```javascript
/*
 * fs.readlink(path, callback)
 * @params path 符号链接路径以及文件名
 * callback @params err 错误对象
 * callback @params linkString [String] 另一个文件的目录、文件名
 */
fs.symlink('./srclink.txt', './symlink.txt', err => {
    if (err) throw err;
    console.log('创建符号链接symlink.txt成功');
    fs.readlink('./symlink.txt', (err, linkString) => {
        if (err) throw err;
        console.log(linkString);
    });
});
```


## Path模块
```javascript
/*
 * const path = require('path')
 * @property path.delimiter 提供平台特定的路径分隔符 win;posix:
 * @property path.posix
 * @property path.sep
 * @property path.win32
 * 
 * @method   path.isabsolute(path) 判定 path 是否为一个绝对路径
 * @method   path.basename(path,[ext]) 返回path的最后一部分，ext为后缀
 * @method   path.dirname(path) 返回path的目录名
 * @method   path.extname(path) 返回path的拓展名
 * 
 * @method   path.format(pathObject) 从对象返回路径字符串
 * @method   path.parse(path) 根据path返回pathObject对象
 * * * * *   pathObject = {dir,root,base,name,ext}
 * * * * *   优先级 dir > root base > name & ext
 * 
 * @method   path.normalize(path) 规范化path 解析'..''.',路径分隔符去重
 * @method   path.join([...path]) 使用平台特定的分隔符把全部给定的 path片段连接到一起，并规范化生成的路径
 * @method   path.relative(fromPath,toPath) 返回从fromPath到toPath的相对路径
 * @method   path.resolve([...path]) 全部给定的path解析为一个绝对路径
 */

// /foo/bar/baz/asdf
let path_normalize = path.normalize('/foo/bar//baz/asdf/quux/..');

// /foo/bar/baz/asdf
let path_join = path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');

// /bar/img/imgage.png
let path_resolve = path.resolve('/foo', 'bar', '/bar/img', '../img/imgage.png');

// ../../impl/bbb
let path_relative = path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb');

// /data/orandea/test/aaa
let path_dirname = path.dirname('/data/orandea/test/aaa/img.gif');

//  img.gif
let path_basename = path.basename('/data/orandea/test/aaa/img.gif');

//  .gif
let path_extname = path.extname('/data/orandea/test/aaa/img.gif');

// { 
//  root: '/',
//  dir: '/data/orandea/test/aaa',
//  base: 'img.gif',
//  ext: '.gif',
//  name: 'img' 
// }
let path_parse = path.parse('/data/orandea/test/aaa/img.gif');

// /data/orandea/test/aaa/img.gif
let path_format = path.format({
    root: '/',
    dir: '/data/orandea/test/aaa',
    base: 'img.gif',
    ext: '.gif',
    name: 'img'
});
```

>原创内容，欢迎交流转载请注明出处