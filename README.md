# HITSZ-safety-education-course-scripts
HITSZ安全教育课程学习脚本

> 声明：本脚本仅做交流学习技术使用，请勿将此脚本用于收费或非法用途。

脚本使用方法：

0. 在电脑的Chrome/Firefox/Edge浏览器打开[课程主页](http://weiban.mycourse.cn)，先使用微信扫码登录。
1. 上面使用的浏览器的**插件商店**内搜索`Tampermonkey`，或者可以查阅[官网](https://www.tampermonkey.net/)获取插件安装方法。
2. 添加脚本
  - 第一种方法：直接添加。复制下面的代码，在tampermonkey插件内点击`添加脚本`，然后粘贴再保存。
  ```javascript
// ==UserScript==
// @name         HITSZ安全教育课程学习脚本
// @namespace    https://www.chiro.work/
// @version      0.1
// @description  HITSZ安全教育课程学习脚本
// @author       Chiro
// @match        http://weiban.mycourse.cn/*
// @require      http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    function getQueryString(name) {
        let querys = window.location.href.split('?')[1].split('&');
        for (let q of querys) {
            let qs = q.split('=');
            if (qs.length <= 1) continue;
            if (qs[0] === name) return qs[1];
        }
        return null;
    }
    function sleep (time) {
        return new Promise((resolve) => setTimeout(resolve, time));
    }
    async function main() {
        await sleep(2000);
        if (window.location.href === 'http://weiban.mycourse.cn/#/') {
            console.log("主页");
            // 进入学习任务选择页面
            $('.mod-more').click();
            setTimeout(onchange, 1000);
        } else if (window.location.href === 'http://weiban.mycourse.cn/#/learning-task-list') {
            console.log("学习任务选择页面");
            // 引导点击下面的两个任务
            $(".mod-headline").text("脚本 ：点击下面的一个学习任务");
            $('.task-block').onclick = () => {
                setTimeout(onchange, 1000);
            };
        } else if (window.location.href === 'http://weiban.mycourse.cn/#/course?projectType=pre' ||
                   window.location.href.startsWith('http://weiban.mycourse.cn/#/course?projectId')) {
            console.log("课程列表页面");
            let stats = $('div.state');
            let found = false;
            for (let i=1; i<stats.length; i++) {
                let sp = stats[i].textContent.split('/');
                if (sp[0] === sp[1]) continue;
                console.log($('h3')[i - 1].textContent, stats[i].textContent);
                $('a.btn')[i - 1].click();
                found = true;
                break;
            }
            if (!found) {
                history.go(-1);
                setTimeout(onchange, 1000);
            }
            setTimeout(onchange, 1000);
        } else if (window.location.href.startsWith('http://weiban.mycourse.cn/#/course/list')){
            console.log('微课列表页面');
            // 请求微课信息
            let formData = {
                str: '',
                append(tag, val) {
                    if (this.str.length !== 0) this.str += '&';
                    this.str += tag + '=' + val;
                }
            };
            try {
                let user = JSON.parse(localStorage.getItem('user'));
                formData.append('userProjectId', getQueryString('projectId'));
                formData.append('chooseType', getQueryString('subjectType'));
                formData.append('categoryCode', getQueryString('categoryCode'));
                formData.append('name', '');
                formData.append('userId', user.userId);
                formData.append('tenantCode', user.tenantCode);
                formData.append('token', user.token);
                // 假装网络不好
                let response = await fetch(`https://weiban.mycourse.cn/pharos/usercourse/listCourse.do?timestamp=${parseInt(new Date().getTime() / 1000) - 10}`, {
                    method: 'POST',
                    body: formData.str,
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                });
                let data = await response.json();
                console.log('got data', data);
                let promises = [];
                let functions = [];
                for (let d of data.data) {
                    if (d.finished != 2) continue;
                    // 请求完成这个微课
                    let formData2 = {
                        str: '',
                        append(tag, val) {
                            if (this.str.length !== 0) this.str += '&';
                            this.str += tag + '=' + val;
                        }
                    };
                    formData2.append('userProjectId', getQueryString('projectId'));
                    formData2.append('userId', user.userId);
                    formData2.append('tenantCode', user.tenantCode);
                    formData2.append('token', user.token);
                    let courseId = d.resourceId;
                    formData2.append('courseId', courseId);
                    console.log('studying', d.resourceName);
                    await fetch(`https://weiban.mycourse.cn/pharos/usercourse/study.do?timestamp=${parseInt(new Date().getTime() / 1000) - 5}`, {
                        method: "POST",
                        body: formData2.str,
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded'
                        }
                    });
                    await sleep(20);
                    console.log('finishing', d.resourceName);
                    await fetch(`https://weiban.mycourse.cn/pharos/usercourse/finish.do?userCourseId=${d.userCourseId}&tenantCode=${user.tenantCode}`, {
                        method: "GET",
                        mode: "no-cors",
                        headers: {
                            "Content-Type": "application/x-www-form-urlencoded"
                        },
                        credentials: "include",
                        Referer: `https://mcwk.mycourse.cn/course/A25001/A25001.html?userCourseId=${d.userCourseId}&tenantCode=${user.tenantCode}&type=1`
                    });
                }
                console.log("完成所有微课");
                history.go(-1);
            } catch(e) {
                console.error("失败", e);
            }
            setTimeout(onchange, 1000);
        } else if (window.location.href.startsWith('http://weiban.mycourse.cn/#/course/detail')) {
        }
    }
    async function onchange() {
        console.log("onchange");
        await main();
    }
    onchange();
})();
  ```
  - 第二种方法，可能需要科学上网。
      1. 打开[这个链接](https://greasyfork.org/zh-CN/scripts/420014-hitsz%E5%AE%89%E5%85%A8%E6%95%99%E8%82%B2%E8%AF%BE%E7%A8%8B%E5%AD%A6%E4%B9%A0%E8%84%9A%E6%9C%AC)
      2. 点击“添加脚本”
3. 打开[课程主页](http://weiban.mycourse.cn)，脚本开始工作。中间有学习任务选择界面需要用户手动点击。


**在遇到卡住不动的时候，请刷新网页。**
