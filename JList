// JavaScript list grid

(function () {


    var g_url; // 数据源URL
    var g_config = g_config_default;
    var g_page; // 当前页
    var g_data;
    var g_table_selector;

    var g_config_default = {};

    function null_func() {}

    /*
     * 创建thead部分
     */
    function create_thead(config) {
        var txt = '<thead><tr>';
        for (var i = 0, column; i < config.columns.length; i++) {
            column = config.columns[i];
            if (i == 0 && column.head == 'check')
                txt += '<th class="row-selected row-selected" width="' + column.width + '"> <input type="checkbox" id="checkAll" class="check-all regular-checkbox"><label for="checkAll"></label></th>';
            else {
                if (column.width == '0%')
                    txt += '<th style="display:none">' + column.head + '</th>';
                else
                    txt += '<th width="' + column.width + '">' + column.head + '</th>';
            }
        }

        txt += '</tr></thead><tbody></tbody>';

        return txt;
    }


    function create_tr(item, config) {
        var columns = config.columns;
        var data_id = item[config.id_field];
        var txt = '<tr data-id="' + data_id + '">';
        var column, enable;

        for (var j = 0; j < columns.length; j++) {
            column = columns[j];

            if (j == 0 && column.field == 'check') {
                // 栏位为选取框
                txt += '<td><input class="ids regular-checkbox" type="checkbox" value="' + item.id + '" name="ids[]" id="check_' + data_id + '"><label for="check_' + data_id + '"></label></td>';
            } else if (column.field == 'options') {
                // 栏位为操作选项
                txt += '<td class="option">';
                for (var k = 0; k < column.options.length; k++) {
                    var option = column.options[k];

                    // 调用传入的enable_handler，检查该option是否enable
                    enable = false;
                    if (option.click_handler != null) {
                        if (option.enable_handler != null)
                            enable = option.enable_handler(item);
                        else
                            enable = true;
                    }

                    if (enable)
                        txt += '<a class="option_' + option.class + '" href="#">' + option.title + '</a>  ';
                    else
                        txt += '<span style="color:#aaa">' + option.title + ' </span>';
                }
                txt += '</td>';
            } else {
                // 正常栏位
                var value = item[column.field];

                if (column.width == '0%') {
                    // 隐藏栏位，用于记录data
                    txt += '<td class="field_' + name[j] + '" style="display:none" data=' + value + '/>';
                } else {
                    var disp_value = value;
                    if (column.display_handler != null) {
                        disp_value = column.display_handler(value);
                    }

                    if (disp_value != null)
                        txt += '<td class="field_' + column.field + ' "data="' + value + '">' + disp_value + '</td>';
                    else
                        txt += '<td class="field_' + column.field + '">' + value + '</td>';
                }
            }
        }
        txt += '</tr>';

        return txt;
    }

    /*
     * 创建tbody部分
     */
    function create_tbody(rows, config) {
        var txt = "";

        if (rows == null)
            return txt;

        for (var i = 0; i < rows.length; i++) {
            txt += create_tr(rows[i], config);
        }

        return txt;
    }


    /*
     * 为options中的操作创建click的handler
     */
    function set_options_click_handler(config) {
        if (config == null || config.columns == null)
            return;

        for (var i = 0, column; i < config.columns.length; i++) {
            column = config.columns[i];

            if (column.field == 'options' && column.options != null) {
                for (var j = 0, option; j < column.options.length; j++) {
                    option = column.options[j];
                    $('.option_' + option.class).click(option.click_handler);
                }
            }
        }
    }

    function set_opetion_hover() {
        $(".data-table tbody tr").hover(
            function () {
                $(this).find('td.option a').css('visibility', 'visible');
                $(this).find('td.option span').css('visibility', 'visible');
            },
            function () {
                $(this).find('td.option a').css('visibility', 'hidden');
                $(this).find('td.option span').css('visibility', 'hidden');
            });
    }

    /*
     * 创建分页导航
     */
    function make_nav_bar(total, current_page) {
        var item = '';
        var page = 0;
        var offset = g_config.rows_of_page;

        for (var i = 0; i < total; i += offset) {
            page++;
            if (page == current_page)
                item += '<span class="current">' + page + '</span>';
            else
                item += '<a class="page-nav num" href="#" data-page="' + page + '">' + page + '</a>';
        }

        var txt = '';
        txt += '<div class="fl"><div class="page" style="margin:0"><div>';
        txt += item;
        txt += '<span class="rows">共 ' + total + ' 条记录</span>'

        txt += '</div></div></div>'

        $('#' + g_config.nav_id).html(txt);

        $('.page-nav').click(function (event) {
            event.preventDefault();
            var page = $(this).attr('data-page');
            load_page(page);
        });
    }


    /**
     * load_grid_data_internal从URL发起AJAX请求的回调函数
     * 返回的数据格式必须为JSON，为{errno, errmsg, data{ total, rows{} } }
     * rows为数据行的数组
     */
    function make_table(data, config) {
        var rows = data.rows;

        g_table_selector.children("tbody").html(create_tbody(rows, config));

        set_options_click_handler(config);

        var total = 0;
        if (data != null)
            total = data.total;
        make_nav_bar(total, g_page);

        set_opetion_hover();
    }


    function create_page_params(page) {
        if (g_config.rows_of_page != null) {
            var start = (page - 1) * g_config.rows_of_page;
            return '&start=' + start + '&offset=' + g_config.rows_of_page;
        }
        else
            return '';
    }


    function load_page( page ){
        if (page != null && page != undefined)
            g_page = page;

        url = g_url + create_page_params(g_page);

        $.pallasAjax.get(url,
            function (json) {
                if (json != null) {
                    g_data = json.data;
                    make_table(g_data, g_config);
                }
            },
            false);
    }


    function load_data( url, page) {


        if( url != null )
            g_url = url;

        load_page( page );
    }

    function init( config, url, load ){

        if (config != null )
            g_config = config;
        else
            g_config = g_config_default;

        var thead = create_thead(g_config);
        g_table_selector = $('#' + g_config.table_id);
        g_table_selector.html(thead);

        if(  url != null ){
            g_url = url;
            if( load == true )
                load_data( url, 1 );
        }

        return this;
    }

    function get_data() {
        return g_data;
    }

    function refresh(){
        make_table(g_data, g_config);
    }

    function get_item( tr ){
        if( g_data == null )
            return null;

        var rows = g_data.rows;

        if( rows == null )
            return;

        var id = tr.attr('data-id');

        for( var i=0; i< rows.length; i++ ){
            if(  rows[i][g_config.id_field] == id )
                return rows[i];
        }

        return null;
    }

    var JGrid = {
        init:init,
        load_data: load_data,
        get_data: get_data,
        get_item: get_item,
        refresh: refresh,
        //get_table_list:get_table_list,
        //search_table:search_table,
        //set_status: set_status
    };

    $.extend({JList: JList});
})
();
