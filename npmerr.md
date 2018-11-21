# npm install Error: EACCES: permission denied

在使用 npm 安装模块时碰到的问题，异常信息如下。

这里简单记录一下解决方法，给命令增加如下的一些参数，大概就是一些允许权限之类的，没有具体研究：
```
npm install --unsafe-perm=true --allow-root
```

**错误信息**
```
[root@weilu125 adminMongo-master]# npm install
npm WARN deprecated to-iso-string@0.0.2: to-iso-string has been deprecated, use @segment/to-iso-string instead.
npm WARN deprecated jade@0.26.3: Jade has been renamed to pug, please install the latest version of pug instead of jade
npm WARN deprecated electron-prebuilt@1.4.13: electron-prebuilt has been renamed to electron. For more details, see http://electron.atom.io/blog/2016/08/16/npm-install-electron
npm WARN deprecated sprintf@0.1.5: The sprintf package is deprecated in favor of sprintf-js.
npm WARN deprecated mongodb@2.2.16: Please upgrade to 2.2.19 or higher
npm WARN deprecated minimatch@0.3.0: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated circular-json@0.3.3: CircularJSON is in maintenance only, flatted is its successor.

> electron-prebuilt@1.4.13 postinstall /usr/local/adminMongo-master/node_modules/electron-prebuilt
> node install.js

/usr/local/adminMongo-master/node_modules/electron-prebuilt/install.js:22
  throw err
  ^

Error: EACCES: permission denied, mkdir '/usr/local/adminMongo-master/node_modules/electron-prebuilt/.electron'
npm WARN mocha-jsdom@1.2.0 requires a peer of jsdom@>10.0.0 but none is installed. You must install peer dependencies yourself.

npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! electron-prebuilt@1.4.13 postinstall: `node install.js`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the electron-prebuilt@1.4.13 postinstall script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2018-11-21T01_22_13_662Z-debug.log
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NDgyNjcwODFdfQ==
-->