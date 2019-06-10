---
title: Worker记录
date: 2019-06-10 17:41:06
tags:
---

使用Worker进行多线程请求数据处理：

````````js
// 创建Worker
if (Worker && fetch) {
    const createWorker = (f) => {
        const blob = new Blob(['(' + f.toString() + ')()']);
        const url = window.URL.createObjectURL(blob);
        const worker = new Worker(url);
        return worker;
    };
    
    let pollingWorker = createWorker(function(e) {
        self.onmessage = function(e) {
            // 获取数据
            fetch(e.data.url, {
                method: 'POST',
                body: e.data.params,
                headers: new Headers({
                    'Accept': 'application/json',
                    'Content-Type': 'application/json',
                    'x-access-token': e.data.SESSIONID
                })
            }).then(function(res) {
                res.json().then(function (data) {
                    self.postMessage(data);
                });
            });
        }
    });

    // 主线程传参
    pollingWorker.postMessage({
        url: window.location.origin + url,
        params: JSON.stringify(params),
        SESSIONID: window.localStorage.getItem('SESSIONID')
    });
    // 主线程接收处理后数据
    pollingWorker.onmessage = (e) => {
        // 关闭线程
        pollingWorker.terminate();
        // 处理
    }
} else {
    // 兼容处理
}

````````