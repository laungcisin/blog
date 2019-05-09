---
title: layui-table-行选中checkbox勾选-行变色
date: 2088-01-01 08:00:02
tags: layui
categories: 前端
---

![layui-table-行选中checkbox勾选-行变色](layui-table-行选中checkbox勾选-行变色.png 'layui-table-行选中checkbox勾选-行变色')


### 完整代码实现
```javascript
    layui.use([..., 'table'], function () {
        var table = layui.table;
        ...
        
        table.render({
             ...
        });
        
        //监听复选框事件，被选中的行高亮显示
        table.on('checkbox(metadataTableFilter)', function (obj) {
            if (obj.checked == true && obj.type == 'all') {
                //点击全选
                $('.layui-table-body table.layui-table tbody tr').addClass('layui-table-click');
            } else if (obj.checked == false && obj.type == 'all') {
                //点击全不选
                $('.layui-table-body table.layui-table tbody tr').removeClass('layui-table-click');
            } else if (obj.checked == true && obj.type == 'one') {
                //点击单行
                if (obj.checked == true) {
                    obj.tr.addClass('layui-table-click');
                } else {
                    obj.tr.removeClass('layui-table-click');
                }
            } else if (obj.checked == false && obj.type == 'one') {
                //点击全选之后点击单行
                if (obj.tr.hasClass('layui-table-click')) {
                    obj.tr.removeClass('layui-table-click');
                }
            }
        });

        ...
    });

    ...
    //单击table行，勾选checkbox事件
    $(document).on("click", ".layui-table-body table.layui-table tbody tr", function () {
        var index = $(this).attr('data-index');
        var tableBox = $(this).parents('.layui-table-box');
        //存在固定列
        var tableDiv;
        if (tableBox.find(".layui-table-fixed.layui-table-fixed-l").length > 0) {
            tableDiv = tableBox.find(".layui-table-fixed.layui-table-fixed-l");
        } else {
            tableDiv = tableBox.find(".layui-table-body.layui-table-main");
        }
        var checkLength = tableDiv.find("tr[data-index=" + index + "]").find(
            "td div.layui-form-checked").length;

        var checkCell = tableDiv.find("tr[data-index=" + index + "]").find(
            "td div.laytable-cell-checkbox div.layui-form-checkbox I");
        if (checkCell.length > 0) {
            checkCell.click();
        }
    });

    $(document).on("click", "td div.laytable-cell-checkbox div.layui-form-checkbox", function (e) {
        e.stopPropagation();
    });
```