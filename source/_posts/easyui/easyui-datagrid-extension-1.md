---
title: 扩展easyui datagrid 编辑网格插件
date: 2022-04-09 16:18:23
tags: easyui
---

# 说明

扩展datagrid 的编辑，实现使用方向来进行表格编辑操作，减少编辑一行就动一下鼠标的麻烦。

## 依赖

从easyui 上扩展的，依赖jquery,easyui 

## GridEdit 扩展内容

### 属性

|   名称    |   类型    |   描述    |   默认值  |
|   ---     |   ---     |   ---     |   ---     |
|   hasDeleteColumn     |   boolean     |   是否在最后一列生成删除操作列    |   true    |
|   deleteColumnWidth   |   int         |   生成删除操作列的宽度    |   50  |
|   primarykey  |   string  |   设置列主键字段  | id    |

### 事件

|   名称    |   参数    |   描述    |
|   ---     |   ---     |   ---     |
|   onDeleteRow     |   value,row,index     |  点击删除按钮时触发，如果是没有主键值则默认删除但是也会触发。<br/> value: 当前行id<br/> row: 当前行json对象<br/> index:当前行索引    |
|   onInitNewRow   |   index         |   新增一行时触发<br/>index: 新增行索引<br/> 返回值： 返回值对象将会用作当前行的对象值，不返回则使用默认的新增空值对象   |

### 方法
|   名称    |   参数    |   描述    |
|   ---     |   ---     |   ---     |
|   endEdit |   index     | index为正在编辑的行， 不传参数则默认结束正在编辑的行  |
|   clear   |   none  | 清空当前表格所有的数据并保留编辑行  |


## 插件代码

```javascript
/*
    扩展datagrid 让其可以通过方向键来进行数据的编辑
    建议 独立新创建一个文件来保存此代码
*/
"use strict";
$(function () {
    $.fn.gridedit = function (options,param) {

        if (typeof options === 'string') {
            if($.fn.gridedit.methods[options]) {
                return $.fn.gridedit.methods[options](this, param);
            } else {
                var $tb = $(this);
                var args = arguments;
                return $tb.datagrid.apply($tb, args);
            }
        }

        options = options || {};
        return this.each(function () {
            var state = $.data(this, 'gridedit');
            var opts;
            if (state) {
                opts = $.extend(state.options, options);
                state.options = opts;
            } else {
                options = $.extend({}, $.fn.gridedit.defaults, options);
                // 扩展默认的属性
                var defaults = {
                    hasDeleteColumn: true, //是否有删除列
                    deleteColumnWidth: 50, //删除列宽
                    onDeleteRow: function (value, row, index) {
                    }, //执行远程删除事件
                    singleSelect: true, //默认单选
                    primarykey: 'id', //默认列的id属性  用于生成删除按钮时使用
                    onInitNewRow: function (index) {
                    }, // 新增行时触发 ， 参数是索引，需要返回一个新增的行对象，用于操作
                    onLoadSuccess: function () {
                    }
                };
                options = $.extend(defaults, options || {});

                opts = options;
                $.data(this, 'gridedit', {options: opts});
            }

            var _grid = {
                //表格的JQuery 对象
                grid: null,
                //当前操作索引
                editIndex: undefined,
                //结束正在编辑的行
                endEditing: function () {
                    if (this.editIndex == undefined) return true;

                    if (this.grid.datagrid('validateRow', this.editIndex)) {
                        this.grid.datagrid('endEdit', this.editIndex);
                        this.renderDeleteButtom();
                        this.editIndex = undefined;
                        return true;
                    } else {
                        return false;
                    }
                },
                //渲染最后一列删除按钮
                renderDeleteButtom: function () {
                    $('.col-delete-btn').linkbutton({text: '删除', plain: true, iconCls: 'icon-remove'});
                },
                //初始化第一可编辑行
                initialBeginEditorRow: function () {
                    var rows = this.grid.datagrid('getRows');
                    if (rows.length == 0) {
                        this.grid.datagrid('appendRow', options.onInitNewRow.call(this.grid, 0) || {});
                        this.grid.datagrid('selectRow', 0).datagrid('beginEdit', 0);
                        this.grid.datagrid('beginEdit', 0);
                        var editors = this.grid.datagrid('getEditors', 0);
                        this.bindEditorsEvent(editors);
                        if (editors.length > 0) {
                            $(editors[0].target).focus();
                        } else {
                            console.error('gridedit 仅支持可编辑的列，需要在列配置editor');
                        }
                        this.editIndex = 0;
                    }
                },
                // 获取当前焦点控件列索引
                getFocusColIndex: function (input) {
                    return $(input).parents('td[field]').index();
                },
                //这里扩展 onRowClick 事件，在保留原有事件的同时 ，增加功能
                appendClickRowEvent: function () {
                    var gridAction = this;
                    var $tb = this.grid;
                    var clickRowBeginEditFunc = function (index, row) {
                        if (gridAction.editIndex != index) {
                            if (gridAction.endEditing()) {
                                $tb.datagrid('selectRow', index).datagrid('beginEdit', index);
                                var editors = $tb.datagrid('getEditors', index);
                                gridAction.bindEditorsEvent(editors);
                                gridAction.editorFocus(editors[0]);
                                gridAction.editIndex = index;
                            } else {
                                $tb.datagrid('selectRow', gridAction.editIndex);
                            }
                        }
                    }
                    if (options.onClickRow) {
                        var oldClickRow = options.onClickRow;
                        options.onClickRow = function (index, row) {
                            oldClickRow.call(this, index, row);
                            clickRowBeginEditFunc.call(this, index, row);
                        }
                    } else {
                        options.onClickRow = function (index, row) {
                            clickRowBeginEditFunc.call(this, index, row);
                        }
                    }
                },
                appendLoadSuccessEvent: function () {
                    var gridAction = this;
                    var $tb = this.grid;
                    if (options.onLoadSuccess) {
                        var oldLoadSucc = options.onLoadSuccess;
                        options.onLoadSuccess = function (index, row) {
                            gridAction.renderDeleteButtom();
                            gridAction.editIndex = undefined;
                            oldLoadSucc.call(this, index, row);
                        };
                    } else {
                        options.onLoadSuccess = function (index, row) {
                            gridAction.renderDeleteButtom();
                            gridAction.editIndex = undefined;
                        }
                    }
                },
                deleteRow: function (row, rowIndex) {
                    var gridAction = this;
                    if (row && row.hasOwnProperty(options.primarykey)) {
                        Msg.confirm('是否永久删除该行？ ', function (isOk) {
                            if (isOk) {
                                options.onDeleteRow.call(gridAction.grid, row[options.primarykey], row, rowIndex);
                                gridAction.grid.datagrid('deleteRow', rowIndex);
                                gridAction.editIndex = undefined;
                                gridAction.initialBeginEditorRow();
                            }
                        });
                    } else {
                        gridAction.grid.datagrid('deleteRow', rowIndex);
                        gridAction.editIndex = undefined;
                        gridAction.initialBeginEditorRow();
                    }
                },
                //根据配置添加最后一行删除列
                appendDeleteColumn: function () {
                    if (options.hasDeleteColumn) {
                        var gridAction = this;
                        var $tb = this.grid;
                        var columns = this.grid.datagrid('options').columns;
                        columns[0].push({
                            field: 'col_delete_btn',
                            title: '操作',
                            width: options.deleteColumnWidth,
                            align: 'center',
                            formatter: function (value, row, index) {
                                return '<a href="javascript:void(0);" data-id="' + row[options.primarykey] + '" class="col-delete-btn" ></a>';
                            }
                        });
                        var $ptb = this.grid.parent();
                        //绑定点击事件
                        $ptb.delegate('a.col-delete-btn', 'click', function () {
                            var $tr = $(this).parents('tr.datagrid-row');
                            var rowIndex = window.parseInt($tr.attr('datagrid-row-index'));
                            var row = $tb.datagrid('getRows')[rowIndex];
                            gridAction.deleteRow(row, rowIndex);
                        });
                        //绑定keydown 事件
                        $ptb.delegate('a.col-delete-btn', 'keydown', function (e) {
                            if (e.key == "ArrowLeft") {
                                var $tr = $(this).parents('tr.datagrid-row-editing');
                                var rowIndex = window.parseInt($tr.attr('datagrid-row-index'));
                                var editors = $tb.datagrid('getEditors', rowIndex);

                                gridAction.editorFocus(editors[editors.length - 1]);

                            } else if (e.key == "ArrowRight") {
                                var $tr = $(this).parents('tr.datagrid-row-editing');
                                var rowIndex = window.parseInt($tr.attr('datagrid-row-index'));
                                var editors = $tb.datagrid('getEditors', rowIndex);
                                gridAction.editorFocus(editors[0]);
                            } else if (e.key == "ArrowUp") {
                                var $mainTable = $(this).parents('table.datagrid-btable');

                                var $tr = $(this).parents('tr.datagrid-row-editing');
                                var rowIndex = window.parseInt($tr.attr('datagrid-row-index'));

                                if ($tb.datagrid('validateRow', gridAction.editIndex) === false) return;

                                var calcIndex = rowIndex - 1;
                                if (calcIndex < 0) return;

                                $tb.datagrid('endEdit', rowIndex);
                                $tb.datagrid('selectRow', calcIndex).datagrid('beginEdit', calcIndex);

                                $tr = $mainTable.find('tr.datagrid-row-editing');

                                rowIndex = window.parseInt($tr.attr('datagrid-row-index'));
                                var editors = $tb.datagrid('getEditors', rowIndex);

                                gridAction.bindEditorsEvent(editors);

                                var $delBtn = $tr.find('>td').eq(editors.length).find('a.l-btn');
                                if ($delBtn.length > 0)
                                    $delBtn.focus();

                                gridAction.editIndex = rowIndex;
                                gridAction.renderDeleteButtom();
                            } else if (e.key == "ArrowDown") {

                                if ($tb.datagrid('validateRow', gridAction.editIndex) === false) return;

                                var $mainTable = $(this).parents('table.datagrid-btable');
                                var $tr = $(this).parents('tr.datagrid-row-editing');
                                var rowIndex = window.parseInt($tr.attr('datagrid-row-index'));

                                var calcIndex = rowIndex + 1;
                                var row = $tb.datagrid('getRows')[calcIndex];
                                if (row) {

                                    $tb.datagrid('endEdit', rowIndex);
                                    $tb.datagrid('selectRow', calcIndex).datagrid('beginEdit', calcIndex);

                                    gridAction.renderDeleteButtom();

                                    $tr = $mainTable.find('tr.datagrid-row-editing');

                                    rowIndex = window.parseInt($tr.attr('datagrid-row-index'));
                                    var editors = $tb.datagrid('getEditors', rowIndex);

                                    gridAction.bindEditorsEvent(editors);

                                    var $delBtn = $tr.find('>td').eq(editors.length).find('a.l-btn');
                                    if ($delBtn.length > 0)
                                        $delBtn.focus();

                                    gridAction.editIndex = rowIndex;
                                }
                            }
                        });
                    }
                },
                bindEditorsEvent: function (editors) {
                    var gridAction = this;
                    var $tb = this.grid;

                    // start keydown
                    function keydown(e) {
                        var isDirection = false;
                        var $tr = $(this).closest("tr.datagrid-row");
                        var rowIndex = window.parseInt($tr.attr('datagrid-row-index'));
                        var calcIndex = -1;
                        switch (e.key) {
                            case "ArrowUp":


                                calcIndex = rowIndex - 1;
                                if (calcIndex < 0) return;
                                isDirection = true;
                                if ($tb.datagrid('validateRow', gridAction.editIndex) === false) return;

                                var focusColIndex = gridAction.getFocusColIndex(this);
                                $tb.datagrid('endEdit', rowIndex);
                                gridAction.renderDeleteButtom();

                                $tb.datagrid('selectRow', calcIndex).datagrid('beginEdit', calcIndex);
                                var editors = $tb.datagrid('getEditors', calcIndex);

                                gridAction.bindEditorsEvent(editors);

                                gridAction.editorFocus(editors[focusColIndex]);

                                gridAction.editIndex = calcIndex;
                                break;
                            case "ArrowDown":

                                isDirection = true;
                                if ($tb.datagrid('validateRow', gridAction.editIndex) === false) return;

                                var focusColIndex = gridAction.getFocusColIndex(this);
                                $tb.datagrid('endEdit', rowIndex);
                                gridAction.renderDeleteButtom();

                                calcIndex = rowIndex + 1;
                                var row = $tb.datagrid('getRows')[calcIndex];
                                if (row) {
                                    $tb.datagrid('selectRow', calcIndex).datagrid('beginEdit', calcIndex);
                                } else {
                                    var newRow = options.onInitNewRow.call($tb, calcIndex);
                                    $tb.datagrid('appendRow', newRow || {});
                                    $tb.datagrid('selectRow', calcIndex).datagrid('beginEdit', calcIndex);
                                }

                                var editors = $tb.datagrid('getEditors', calcIndex);

                                gridAction.bindEditorsEvent(editors);

                                gridAction.editorFocus(editors[focusColIndex]);

                                gridAction.editIndex = calcIndex;
                                break;
                            case "ArrowLeft":
                                var editors = $tb.datagrid('getEditors', rowIndex);
                                var colIndex = gridAction.getFocusColIndex(this);
                                var editor = editors[colIndex - 1];
                                if (editor) {
                                    gridAction.editorFocus(editor);
                                }
                                isDirection = true;
                                break;
                            case "ArrowRight":

                                var editors = $tb.datagrid('getEditors', rowIndex);
                                var colIndex = gridAction.getFocusColIndex(this);
                                var colIdx = colIndex + 1;
                                var editor = editors[colIdx];
                                if (editor) {
                                    gridAction.editorFocus(editor);
                                } else {
                                    var $delBtn = $tr.find('>td').eq(colIdx).find('a.l-btn');
                                    if ($delBtn.length > 0)
                                        $delBtn.focus();
                                }

                                isDirection = true;
                                break;
                            case "Enter":
                                var editors = $tb.datagrid('getEditors', rowIndex);
                                var colIndex = gridAction.getFocusColIndex(this);
                                var colIdx = colIndex + 1;
                                var editor = editors[colIdx];
                                if (editor) {
                                    gridAction.editorFocus(editor);
                                } else {
                                    var $delBtn = $tr.find('>td').eq(colIdx).find('a.l-btn');
                                    if ($delBtn.length > 0)
                                        $delBtn.focus();
                                }
                                isDirection = true;
                                break;
                            case "Delete":
                                var row = gridAction.grid.datagrid('getRows')[rowIndex];
                                gridAction.deleteRow(row, rowIndex);
                                break;
                        }
                        if (isDirection) {
                            e.stopPropagation();//阻止事件冒泡  ，可阻止父类事件的发生
                            e.preventDefault();//阻止默认行为  如A标签
                        }
                    }

                    // end keydown

                    for (var index in editors) {
                        var editor = editors[index];
                        this.bindEditorKeydownEvent(editor, keydown);
                    }
                },
                bindEditorKeydownEvent: function (editor, keydown) {
                    if (editor.type == 'checkbox') {
                        $(editor.target).keydown(keydown);
                    } else {
                        var $target = $(editor.target[editor.type]('textbox'));
                        $target.unbind('keydown');
                        $target.keydown(keydown);
                    }
                },
                editorFocus: function (editor) {
                    if (editor.type == 'checkbox') {
                        $(editor.target).focus();
                    } else {
                        $(editor.target[editor.type]('textbox')).focus();
                    }
                },
                init: function (dom) {
                    this.grid = $(dom);
                    this.appendClickRowEvent();
                    this.appendLoadSuccessEvent();
                    var $grid = this.grid.datagrid(options);
                    this.appendDeleteColumn();
                    this.initialBeginEditorRow();
                    return $grid;
                }
            };

            //对外开放的函数
            var gridMethods = {
                //清空函数
                clear: function() {
                    _grid.grid.datagrid('loadData', {total: 0 , rows: []});
                    _grid.initialBeginEditorRow();
                },
                endEdit: function (index) {
                    _grid.grid.datagrid('endEdit', index || _grid.editIndex);
                    _grid.editIndex = undefined;
                }
            };
            opts.__grid = _grid;
            opts.gridMethods = gridMethods;

            _grid.init(this);
        });
    };

    //扩展fn 函数
    $.fn.gridedit.methods = {
        clear: function (jq, param) {
            return jq.each(function(){
                var ge = $.data(this, 'gridedit');
                ge.options.gridMethods.clear();
            });
        },
        endEdit: function (jq,param) {
            return jq.each(function () {
                var ge = $.data(this, 'gridedit');
                ge.options.gridMethods.endEdit(param);
            });
        }
    };

});
```

## 使用示例

```html

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title></title>
    <link rel="stylesheet" type="text/css" href="https://www.jeasyui.com/easyui/themes/default/easyui.css">
    <link rel="stylesheet" type="text/css" href="https://www.jeasyui.com/easyui/themes/icon.css">
    <link rel="stylesheet" type="text/css" href="https://www.jeasyui.com/easyui/themes/color.css">
    <link rel="stylesheet" type="text/css" href="https://www.jeasyui.com/easyui/demo/demo.css">
    <script type="text/javascript" src="https://code.jquery.com/jquery-1.9.1.min.js"></script>
    <script type="text/javascript" src="https://www.jeasyui.com/easyui/jquery.easyui.min.js"></script>

    <!-- 这里我将插件代码 保存到另一个文件 并其名为 jquery.easyui.griedit.js ，然后引用 -->
    <script type="text/javascript" src="jquery.easyui.gridedit.js"></script>

</head>
<body>
<h2>可编辑表格</h2>
<table id="tb" ></table>
<script type="text/javascript">
    $(function(){
		
		var $tb = $('#tb');
        var cols = [[{
				field: 'text', 
				title: '文本',
				width: 200,
				editor: 'text'
			},{
				field: 'name',
				title: '名称',
				width: 200,
				editor: 'text'
			},{
				field: 'field_1',
				title: '字段1',
				width: 200,
				editor: 'datebox'
			},{
				field: 'field_2',
				title: '字段2',
				width: 200,
				editor: 'combobox'
			}]];

            // 使用的方式和 datagrid 一样 只不过将datagrid 改为了 gridedit 

            $tb.gridedit({
                columns: cols,
                onDeleteRow: function(val, row,index) {
                    console.log(row);
                    console.log('on delete row');
                },
                onClickRow: function(index, row) {
                    console.log('on click row');
                    console.log(this);
                },
                onInitNewRow: function(index) {
                    console.log('on init new row'+ index);
                    return { name:'name '+ (index +1)  };
                }
            });

            // 生成假数据
            var fakeData = [];
            for(var i = 0; i< 10;i++) 
                fakeData.push({ name: 'name ' + i});
            // 加载假数据
            $tb.gridedit('loadData', {total: fakeData.length, rows: fakeData});

            // 调用方法 示例
            $tb.grededit('clear');
            $tb.grededit('endEdit');
});

</script>
 
</body>
</html>

```
