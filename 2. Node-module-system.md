Node có 1 cộng đồng khá lớn và được nhiều người contribute các module, thấy cái nào cần mà có thì lấy về xài khỏi phải viết. Những module này là những đoạn JS như mình viết bth thôi. Tải về nó sẽ nằm trong 1 folder mình gọi ra xài là được.

Mình cũng có thể tạo ra các module rồi up lên cho người khác xài: https://npmjs.org/.
# Installing Package
Cú pháp: 
1. ```npm install package_name``` => latest version
2. ```npm install package_name@x.y.z``` => version x.y.z
3. ```npm install package_name@1.0.x``` => version lastest of 1.0
4. ```npm install package_name@"[>/>=/</<=x.y.z]||..."```
5. ```npm install url```

Tất cả module được cài sẽ nằm ở root/node_modules, muốn xem nhanh thì gõ ```npm ls```.

# Globak package
install bình thường là để xài cho project hiện tại thôi. Còn cái nào mà hầu như project nào cũng xài thì nên cài global.

Cú pháp:
```npm install -g/--global package_name``` =>cài module

Bọn này sẽ được để ở 1 nơi cũng có tên là node_modules. Xem location bằng cách gõ: ```npm root -g```
# Linking package
Dùng để link local module. Giả sử ở module B cần dùng module A.
1. Đứng ở A gõ ```npm link```
2. Đứng ở B gõ ```npm link A```

Để unlink thì làm ngược lại
1. Đứng ở B gõ ```npm unlink A```
2. Sang A gõ ```npm unlink```
# Updating package
1. Kiểm tra outdated package: ```npm outdated [-g]```
2. Update package: ```npm update [-g]``` hoặc ```npm update [-g] package_name```

# Require()
Sau khi install module thì cần gọi ra để xài. Hàm ```require()``` có 1 arg là module cần load. Trả về object để dùng or throw err.

### Core modules
Ông này là module được combined trong Node binary. Có độ ưu tiên cao nhất. Nếu mà có nhiều module cùng tên nó sẽ ưu tiên core module. ex: ```require("http")```
### File modules
Là những module còn lại, dùng absolute/relative path or từ node_modules folder. ex: ```require("a/b/fileX")``` ```require("moduleX")```

# File Extension Processing
Nếu ```require()``` không tìm thấy thì trước khi quăng err nó sẽ thử thêm extension vào coi có không. bao gồm **.js, .json, .node**.

Muốn custome thì dùng
```javascript 
require.extensions[".javascript"] = require.extensions[".js"]
```
Khi require nó sẽ chạy 1 handle, muốn custome thì
```javascript 
require.extensions[".javascript"] = function() { 
    console.log(arguments); 
};
```
# Module location
Sử dụng ```require.resolve("module_name")``` để lấy path tới module.
# Caching
Khi 1 module đã được load thành công nó sẽ cache trong ```require.cache``` với **key là path**
# Package.json
npm sử dụng file này để quản lý dependencies
```javascript
{
 "name": "package-name",
 "version": "0.0.0",
 "main": "index.js",
 "description": "This is a description of the module",
 "keywords": [
  "foo",
  "bar",
  "baz",
 ],
 "author": {
  "name": "Phucnh",
  "email": "phucnh@domain.com",
 "url": "http://www.url1.com"
},
"contributors": [
 {
  "name": "X Contributor",
  "email": "x@domain.com",
  "url": "http://www.url.com"
 }
 ],
 "dependencies": {
  "commander": "1.1.x"
 },
 /*Only dev mode:
  npm install mocha --save-dev
 */
 "devDependencies": {
  "mocha": "~1.8.1"
 },
 "engines": {
  "node": ">=0.10.12",
  "npm": "1.2.x"
 },
 /*mapping of npm commands to script commands*/
 "scripts": {
 "start": "node server.js",
 "test": "echo \"Error: no test specified\" && exit 1"
}
}
```
Thực ra gõ lệnh cho nó tự tạo chứ không cần nhớ, hiểu ý nghĩa các fields trong đó là được:
```npm init```

# Module object
Module là global object, ở file nào cũng truy xuấy được. Nó có 1 field là exports, value của nó được return khi gọi ```require()``` function.

Nếu module đó chỉ expose 1 hàm thì gán thẳng luôn
```javascript
module.exports = function bar() {
 console.log("Inside of bar() function");
}
---
var bar = require("./bar");
console.log(bar);
bar();
```
Còn muốn nhiều thì gán kiểu khác:
```javascript
module.exports.bar = function bar() {
 console.log("Inside of bar() function");
}
---
var bar = require("./bar");
console.log(bar);
bar.bar().
```
> Nhớ là đừng gán ```module.exports = {object}``` là nó mất những cái khác đấy. Ghi đè mà.
### Additional properties
Props | Des
--- | ---
id | An identifier for the module. Typically this is the fully resolved filename of the module
filename | The fully resolved filename of the module
loaded | A Boolean value representing the module's state. If the module has finished loading, this will be true. Otherwise, it will be false.
parent | An object representing the module that loaded the current module.
children | An array of objects representing the modules imported by the current module
