// ==UserScript==
// @name         tapd
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  tapd notification
// @author       hugh
// @match        https://www.tapd.cn/49815309/bugtrace/*
// @exclude      https://www.tapd.cn/49815309/bugtrace/bugs/view*
// @grant        none
// ==/UserScript==

(function () {
    'use strict';
    window.qinsilkTapdConfig = {
        showHasBody: true,
		supportFieldName: ['ID', 'TB开发任务ID', '标题', '严重程度', '优先级', '状态', '处理人', '开发人员'],
		supportFieldFormat: {
			'标题': function(el) {
				return $.trim($(el).find('div a').attr('title'));
			},
			'开发人员': function(el) {
				return $.trim($(el).find(':last-child').html()).replace(' &nbsp;', '');
			}},
        filedIndex:{},
        refreshTime: 180,// 秒,默认180s
        nameFormat : {
        },
        _filterType: {'TYPE_NOT': 'not'},
        filter:{}
    };

    window.NotificationHandler = {
        isNotificationSupported: 'Notification' in window,
        isPermissionGranted: function () {
            return Notification.permission === 'granted';
        },
        requestPermission: function () {
            if (!this.isNotificationSupported) {
                alert('the current browser does not support Notification API');
                return;
            }

            Notification.requestPermission(function (status) {
                var permission = Notification.permission;
            });
        },
        showNotification: function (title, body) {
            if (window.qinsilkTapdConfig.showHasBody) {
                if (!body) {
                    return;
                }
            }
            if (!this.isNotificationSupported) {
                alert('当前浏览器不支持桌面通知！');
                return;
            }
            if (!this.isPermissionGranted()) {
                console.log('获取桌面通知授权失败，请允许桌面通知！');
                return;
            }

            var n = new Notification(title, {
                icon: 'http://web.qinsilk.com/is/static/favicon.ico',
                body: body
            });

            n.onshow = function () {
                //5秒后关闭消息框
                setTimeout(function () {
                    n.close();
                }, 5000);
            };

            n.onclick = function () {
                window.focus();
                alert(window.qinsilkTapdConfig.allBody)
                n.close();
            };

            n.onerror = function () {};
            n.onclose = function () {};
        }
    };
    var getCnName = function(v) {
        if (window.qinsilkTapdConfig) {
            for (var n in window.qinsilkTapdConfig.nameFormat) {
                if (v.indexOf(n) != -1) {
                    return window.qinsilkTapdConfig.nameFormat[n];
                }
            }
        }
		return v;
	};
    window.getItems = function() {
        var tableH = {};
        $($('thead tr')[0]).find('th').each(function (index) {
            var name = $.trim($(this).find(':last-child').html());
            for (var i in window.qinsilkTapdConfig.supportFieldName) {
                if (name.indexOf(window.qinsilkTapdConfig.supportFieldName[i]) == 0) {
                    tableH[index] = window.qinsilkTapdConfig.supportFieldName[i];
                    window.qinsilkTapdConfig.filedIndex[window.qinsilkTapdConfig.supportFieldName[i]] = index;
                    break;
                }
            }
        });
        var items = [];
        $($('tbody')).find('tr').each(function (index) {
            var item = {};
            if (index > 0) {
                $(this).find('td').each(function (index) {
                    if (tableH[index] != null && tableH[index].length > 0) {
                        if (window.qinsilkTapdConfig.supportFieldFormat[tableH[index]] && typeof window.qinsilkTapdConfig.supportFieldFormat[tableH[index]] == 'function') {
                            item[index] = window.qinsilkTapdConfig.supportFieldFormat[tableH[index]](this);
                        } else {
                            item[index] = $.trim($(this).find(':last-child').html());
                        }
                    }
                })
                items.push(item);
            }
        })
        return {table: tableH, items: items};
    }

    $(window).ready(function () {
        if (window.NotificationHandler.isNotificationSupported && !window.NotificationHandler.isPermissionGranted) {
            window.NotificationHandler.requestPermission();
        }
        var formatTask = function(text, name, item, moreMsg){
            if (moreMsg) {
                text += '\n' + getCnName(name) + ': ' + item.taskCount;
                for (var k in item.level) {
                    text += ' ,' + (k == '--' ? '无' : k) + ':' + item.level[k];
                }
            } else {
                text += '\n' + getCnName(name) + ': ' + item.taskCount;
            }
            return text;
        }
        var items = window.getItems();
        var bugCount = Number($.trim($('#all_bug_count').text()));
        window.qinsilkTapdConfig.bugCount = bugCount;
        window.qinsilkTapdConfig.table = items.table;
        window.qinsilkTapdConfig.items = items.items;
        // 进行对比
        var oldTapdItems = window.localStorage.getItem('tapdItems');
        var oldbugCount = window.localStorage.getItem('bugCount') | 0;
        if (oldTapdItems && oldTapdItems.length > 0) {
        oldTapdItems = JSON.parse(oldTapdItems);
        } else {
        oldTapdItems = [];
        }
        var handlerTask = {};
        for(var i in window.qinsilkTapdConfig.items) {
            var itme = window.qinsilkTapdConfig.items[i];
            var isFilter = false;
            if (window.qinsilkTapdConfig.filter && Object.getOwnPropertyNames(window.qinsilkTapdConfig.filter).length > 0) {
                for(var f in window.qinsilkTapdConfig.filter) {
					// 说明有多组过滤规则
                    if (isFilter) {
						isFilter = false;
					}
                    var fval = itme[window.qinsilkTapdConfig.filedIndex[f]];
                    var type = window.qinsilkTapdConfig.filter[f].type;
					$.each(window.qinsilkTapdConfig.filter[f].val, function() {
                        if (type == window.qinsilkTapdConfig._filterType.TYPE_NOT) {
                            if (fval.indexOf(this) == -1) {
                                isFilter = true;
                                return false;
                            }
                        } else {
                            if (fval.indexOf(this) != -1) {
                                isFilter = true;
                                return false;
                            }
                        }
					})
					// 其中一组规则不通过，就退出
					if (!isFilter) {
						break;
					}
                }
            } else {
                isFilter = true;
            }
            var handler = itme[window.qinsilkTapdConfig.filedIndex['处理人']];
            var level = itme[window.qinsilkTapdConfig.filedIndex['严重程度']];
            if (isFilter) {
                if (handlerTask[handler]) {
                    handlerTask[handler].taskCount++;
                } else {
                    handlerTask[handler] = {};
                    handlerTask[handler].isFilter = isFilter;
                    handlerTask[handler].taskCount = 1;
                }
                handlerTask[handler].level = handlerTask[handler].level || {};
                if (handlerTask[handler].level[level]) {
                    handlerTask[handler].level[level]++;
                } else {
                    handlerTask[handler].level[level] = 1;
                }
            }
        }
        var allBody = '';
        var body = '';
        for(var k in handlerTask) {
            if (k.length == 0){
                continue;
            }
            if (handlerTask[k].isFilter){
               body = formatTask(body, k, handlerTask[k]);
            }
            allBody = formatTask(allBody, k, handlerTask[k], true);
        }
        if (handlerTask['']) {
             if (handlerTask[''].isFilter){
               body = formatTask(body, '未 认 领', handlerTask['']);
            }
            allBody = formatTask(allBody, '未 认 领', handlerTask[''], true);
        }
        if (oldbugCount != bugCount) {
            if (oldbugCount > bugCount) {
                window.NotificationHandler.showNotification('减少缺陷数：' + (oldbugCount - bugCount) + ',总数' + bugCount, body);
            } else {
                window.NotificationHandler.showNotification('新增缺陷数：' + (bugCount - oldbugCount) + ',总数' + bugCount, body);
            }
        } else {
            window.NotificationHandler.showNotification('无新增缺陷，总数：' + bugCount, body);
        }
        window.qinsilkTapdConfig.body = body;
        window.qinsilkTapdConfig.allBody = allBody;
        // 保存原有的配置
        window.localStorage.setItem('tapdItems', JSON.stringify(window.qinsilkTapdConfig.items));
        window.localStorage.setItem('bugCount', JSON.stringify(window.qinsilkTapdConfig.bugCount));
        window.localStorage.setItem('body', JSON.stringify(window.qinsilkTapdConfig.body));
    });

    var refreshTime = (window.qinsilkTapdConfig.refreshTime || 60) * 1000;
    setTimeout(function(){
        window.location.reload();
    }, refreshTime);
})();
