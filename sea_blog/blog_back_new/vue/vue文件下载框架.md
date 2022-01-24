---
title: vue文件下载框架
date: 2021-03-01 15:09:19
categories:  [vue]
tags: [vue,fastapi]
---


<!--more-->


#  vue文件下载框架

后端
```python
# -*- coding: UTF-8 -*-

# pip3 install aiofiles==0.5.0

from fastapi import FastAPI
from starlette.responses import FileResponse


app = FastAPI()


@app.get("/file")
def file():
    return FileResponse(file_location, media_type='application/octet-stream',filename=file_name)


if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='127.0.0.1', port=8002)
```



前端
```js
// axios.post('/api/downloadFile',{'file':item}).then(res=>{
//                   let blob = new Blob([res.data], { type: res.headers['content-type'] });
//                   let link = document.createElement('a');
//                   link.href = window.URL.createObjectURL(blob);
//                   link.download =item.slice(item.lastIndexOf('/')+1);
//                   link.click()
//               }).catch(err=>{

//               })
export function downloadFile(res, fileName) {
  if (!res) {
    return
  }
  if (window.navigator.msSaveBlob) {  // IE以及IE内核的浏览器
    try {
      window.navigator.msSaveBlob(res, fileName)  // res为接口返回数据，这里请求的时候已经处理了，如果没处理需要在此之前自行处理var data = new Blob([res.data]) 注意这里需要是数组形式的,fileName就是下载之后的文件名
      // window.navigator.msSaveOrOpenBlob(res, fileName);  //此方法类似上面的方法，区别可自行百度
    } catch (e) {
      console.log(e)
    }
  } else {
    let url = window.URL.createObjectURL(new Blob([res])) // post 下来的res
    let link = document.createElement('a') // 创建a标签
    link.style.display = 'none'
    link.href = url   // 下载文件的地址
    link.setAttribute('download', fileName)// 文件名
    document.body.appendChild(link)
    link.click()  // 点击
    document.body.removeChild(link) // 下载完成移除元素
    window.URL.revokeObjectURL(url) // 释放掉blob对象
  }
}
```
创建Bolb对象
https://segmentfault.com/a/1190000020540788