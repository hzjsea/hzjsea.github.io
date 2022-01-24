---
title: fastapi定时器实践
categories:
  - fastapi
tags:
  - fastapi
abbrlink: a99980f
date: 2020-07-08 12:54:19
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# fastapi定时器实践



## apscheduler+fastapi事件完成
```py
# main.py
from fastapi import FastAPI
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.executors.pool import ThreadPoolExecutor
import os

app = FastAPI()

# 根目录
@app.get("/")
async def hello_world():
    return "hello world"

def func():
    res = os.popen("cat hello").read()
    print(res)

# 创建执行器
executors = {
    'default': ThreadPoolExecutor(20),
}
# 创建定时任务的调度器对象
scheduler = BackgroundScheduler(executors=executors,max_instances=100)
scheduler.add_job(func,'interval',seconds=10,id="helloworld")

# 返回一个计数器
@app.post("/update/count")
async def return_counter(nums):
    os.popen("echo {} > hello".format(nums))
    return nums

# 应用开始后，打开计时任务
@app.on_event("startup")
async def start_task():
    # 创建定时任务的调度器对象
    scheduler.start()

# 应用结束后，关闭计时任务
@app.on_event("shutdown")
async def stop_task():
    scheduler.stop()


# 在应用关闭的时候，会执行
@app.on_event("shutdown")
async def shutdown_event():
    items['shutdown'] = {"status":"shutdown"}
    print(items['shutdown'])

# 修改定时器的时间
@router.post("/post/scheduler")
def change_schedule_time(*,time:str):
    """修改定时器的时间
    """
    scheduler.reschedule_job('helloworld', func_get_link_total(), trigger='interval', seconds=int(time))
    return rp.message_success(msg="状态更新成功",data="None")
```