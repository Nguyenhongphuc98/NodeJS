
Thứ quan trọng nhất trong model này là Node chạy **signal thread** - nghĩa là tại 1 thời điểm chỉ có duy nhất 1 task được thực hiện. Nhưng nếu chỉ dừng lại ở đó thì chả ai xài cả đâu. Đi interview mấy ông còn chém banh xác các vấn đề nào là concurrent, parallel. Để có thể tạo ra thứ gọi là `multithreaded` như mấy ông khác thì nó sinh ra khái niệm **EventLoop**. Sẽ có 1 queue các task, công việc của nó là dequeue and enqueue. Có event thì đẩy vào, lấy task ra làm, nếu task đó sinh ra event khác thì cứ đẩy vào cuối tiếp. Khi 1 event xử lý xong thì sẽ trả lại quyền xử lý cho event loop. Nghĩa là thực tế đây không phải multithreaded, nếu có 1 task infinite loop không return control thì task tiếp theo k thể execute.

# Asynchronous Programming
Hầu hết mọi thứ trong nodeJs đều xài async *(1 code block được gọi nhưng không cần chờ thực thi xong để chạy đoạn code tiếp theo)*. Nó cũng như sync function nhưng thêm từ khóa async trước cho đỡ nhầm lẫn. Nó có thêm 1 param là 1 `callback/continuation` để khi async code chạy xong sẽ gọi.
Callback fn nếu có thì nó là param cuối cùng, và cb fn có 2 tham số ```(err, data)```.

### Callback Hell
Khi mà mình có quá nhiều cb fn lồng nhau quá thì gọi là callback hell > khó đọc, khó maintain.
```javascript
var fs = require("fs");
var fileName = "foo.txt";
fs.exists(fileName, function(exists) {
 if (exists) {
    fs.stat(fileName, function(error, stats) {
        if (error) {
            throw error;
        }
        if (stats.isFile()) {
            fs.readFile(fileName, "utf8", function(error, data) {
                if (error) {
                    throw error;
                }
                console.log(data);
            });
        }
    });
 }
});
```
Để tránh callback hell thì cách dễ nhất là viết nhiều hàm nhỏ, mỗi cái handle 1 tý với 1 cái tên dễ xài.

### Exception Handling
Trong sync code thì JavaScript hỗ trợ try-catch. Nhưng mà callback-driven của node thì cho phép excute out side error handling, tại thời điểm nó chạy thì try-catch statement không còn nằm trong call stack. Nên bắt ở đó là vô nghĩa

```JavaScript
var fs = require("fs");
try {
    // empty file to sure it cause err file not found ^^
    fs.readFile("", "utf8", function(error, data) {
        if (error) {
            throw error;
        }
        console.log(data);
    });
} catch (exception) {
    console.log("The exception will not cause here :V:v !")
}
```
Vậy thì cách handle đầu tiên là handle trong fn có err argument.

Cách thứ 2 là dùng global event handler. Node có 1 global object gọi là ```process```
```JavaScript
process.on("uncaughtException", function(error) {
 console.log("The exception was caught!")
});
```
# The async Module
Là một open source module dùng để quản lý sync control flow
### Executing in Series
Bình thường mà viết nhiều async code để nó chạy theo đúng thứ tự, lại dễ đọc là hơi căng. Nhưng với Async module thì chỉ là ```series()``` method.
Nguyên lý hoạt động của hàm này là nhận vào 1 mảng các fn cần chạy tuần tự. Mỗi hàm này có 1 ```callback(err, data)``` . Nếu không có err thì sẽ dùng data làm return value rồi call next fn. Còn mà có lỗi thì out game chứ k xử lý fn tiếp theo. Param thứ 2 của ```series()``` là final callback(err,[result]). Nếu any fn err thì call thẳng đến final callback.
```JavaScript
async.series([
    function(callback) {
        setTimeout(function() {
            console.log("Task 1");
            callback(new Error("Problem in Task 1"), 1);
        }, 200);
    },
    function(callback) {
        setTimeout(function() {
            console.log("Task 2");
            callback(null, 2);
        }, 100);
    }], 
    function(error, results) {
        if (error) {
            console.log(error.toString());
        } else {
        console.log(results);
        }
    }
);

// out 1: 
// Task 1
// Error: Problem in Task 1

// out2: (no throw err)
// Task 1
// Task 2
// [ 1, 2 ]
```

### Executing in Parallel
Thực ra Js vẫn là single thread nên k có parallel thực sự ở đây.
Nhưng mà ```Async``` cung cấp ```parallel()``` để chạy, nó giống ```series()``` nhưng mà không chờ từng cái xong mới tới cái tiếp theo. Về cơ bản là chả khác gì code gọi nhiều hàm thông thường, chỉ có điều là giờ mình có 1 callback để nhận data khi mà tất cả fns đó excute xong.
Muốn limit số lượng fn *parallel* trong 1 thời điểm thì dùng ````parallelLimit()```.

### The Waterfall Model
Nó là 1 series modal mà để thực thi các task theo thứ tự + task sau cần kết quả task trước.
Cách dùng y chang 2 thằng trên chỉ có là 
1. fns phải là array
2. chỉ có data của last task pass to final callback.
3. task fn có thêm addition arguments cung cấp bởi task trước.

```JavaScript
var async = require("async");
async.waterfall([
    function(callback) {
        callback(null, Math.random(), Math.random());
    },
    function(a, b, callback) {
        callback(null, a * a + b * b);
    },
    function(cc, callback) {
        callback(null, Math.sqrt(cc));
    }
    ], function(error, c) {
        console.log(c);
    }
);
```
### The queue model
Queue modal cho phép thực thi các task theo queue. Các task này hoàn toàn có thể add vào bất cứ lúc nào (dynamically). Phù hợp với bài toán producer-consumer.

```javascript
var async = require("async");
var queue = async.queue(function(task, callback) {
    // process the task argument
    console.log(task);
    callback(null);
}, 4);
```

task handler chứa 2 params:
- `task` : object define the task
- `callback` : shoud call with an error once task is processed
Level của queue dùng để chỉ số task được excute đồng thời.

Sau khi queue setup, có thể thêm task vào bằng `push()/unshift()` với task hoặc [task] và optinal callback

```javascript
var i = 0;
setInterval(function() {
    queue.push({
        id: i
    }, function(error) {
        console.log("Finished a task");
    });
    i++;
}, 200);
```
Control num of task at runtime
```javascript
if (queue.length() > threshold) {
    queue.concurrency = 8;
}
```

```javascript
queue.saturated = function() {
    // queue length equal to it's concurrency
    console.log("Queue is saturated");
};
queue.empty = function() {
    // last task is removed from the queue
    console.log("Queue is empty");
};
queue.drain = function() {
    // last task completely processed
    console.log("Queue is drained");
};
```

### Repeating Methods
Repeatly call a function until condition match. Giống với js methd thôi nhưng mà nó chạy async, không block thread.

```javascript
var async = require("async");
var i = 0;
async.whilst(function() {
    // sync truth test
    return i < 5;
}, function(callback) {
    // excute when test true
    setTimeout(function() {
        console.log("i = " + i);
        i++;
        callback(null);
    }, 1000);
}, function(error) {
    console.log("Done!");
});
```

Some other method:
`async.doWhilst(body, test, callback)`
`async.until(test, body, callback)`
`async.doUntil(body, test, callback)`
`memoize(), unmemoize(), each(), map(), filter(), reduce(), some(), and every()`
