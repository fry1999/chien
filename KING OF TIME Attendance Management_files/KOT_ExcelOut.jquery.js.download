/**
 * 表示しているHTMLをxlsとして保存して出力
 */
;(function($) {
    
    "use strict";
    
    $.KOT_ExcelOut = function(target, options) {this.init(target, options); };
    $.extend($.KOT_ExcelOut.prototype, {
        
        settings : {
            //actionTagetは必ず必要
            backgroundTarget : "",
            //企業ID
            companyId : "",
            //管理者ID
            administratorId : "",
            //従業員ID
            employeeId : "",
            //現在のページID
            currentPageId : "",
            //excelからremoveするセレクタ
            removeTargetSelector : [],
            //数値を文字列として出力するかどうか("0"しない "1"する)
            numberToString : "0",
            //使用する文言
            messages: {
                loading: "Loading...",
                download: "Download",
                error: "Error Occurred"
            }
        },
        CONSTS : {
            REGS : [/^-?[0-9]+:[0-9]+$/, /^-?[0-9]+$/, /^-?[0-9]+\.[0-9]+$/],
            REG_FORMAT_TEXT : /^[=＝]/
        },
        exportedFile : null,

        //初期化処理
        init : function(target, options){
            $.extend(this.settings, options);
            this.targetElement = $(target);
            
            //初期化処理が終わるまではブロックする
            this.targetElement.prop("disabled", true);
            this.settings.messages.normal = this.targetElement.text();
            this.initParameters();
            var self = this;
            
            //ダウンロードの際に利用するリンク
            this.downloadLink = $("<a>").css("display", "none");
            this.targetElement.after(this.downloadLink);
            
            $(function() {
                
                self.onExecute(false);
                
                setTimeout(function() {
                    self.blocking = false;
                    self.targetElement.prop("disabled", false);
                }, 5000);
            });
        },
        initParameters : function() {
            //ブロッキング
            this.blocking = false;
            //displayNoneになっている要素を配列で保持
            this.unDisplayList = [];
            //type='text/css'のLINKタグ内のhrefを配列で保持
            this.cssLinkList = [];
            //STYLEタグ内に記述されたCSSテキスト
            this.styleList = [];
            //外部CSSから取得したCSSテキスト
            this.outerStyleList = [];
            //EXCELとして保存するHTMLエレメント（jQueryオブジェクト）
            this.excelElement = {
                html : null,
                head : null,
                body : null
            };
        },
        onExecute : function(isClear) {
            var self = this;
            if(isClear) {
                this.targetElement.off(".excel_out");
                this.downloadLink.attr("href","#");
                this.exportedFile = null;
                this.clearBlock();
                this.targetElement.text(this.settings.messages.normal);
            }
            this.targetElement.on("click.excel_out",function(e){
                if(self.blocking) {
                    e.stopPropagation();
                    return;
                }
                $(this).text(self.settings.messages.loading);
                $(this).prop("disabled", true);
                self.blocking = true;
                self.execute();
                e.stopPropagation();
            });
        },
        onDownload : function() {
            var self = this;
            this.targetElement.off(".excel_out");
            this.clearBlock();
            this.targetElement.text(this.settings.messages.download);
            this.targetElement.on("click.excel_out", function(e) {
                if (navigator.userAgent.indexOf("MSIE ") > -1
                    || navigator.userAgent.indexOf("Trident/") > -1
                    || navigator.userAgent.indexOf("Edge/") > -1){
                    window.navigator.msSaveBlob(self.exportedFile, self.getDownloadFileName());
                } else {
                    var fileURL = URL.createObjectURL(self.exportedFile);
                    self.downloadLink.prop('href',fileURL);
                    self.downloadLink.prop('download',self.getDownloadFileName());
                    self.downloadLink.prop('target','_blank');
                    self.downloadLink[0].click();
                }
                setTimeout(function() {
                    self.onExecute(true);
                },10000);
            });
        },
        clearBlock : function() {
            this.blocking = false;
            this.targetElement.prop("disabled", false);
        },
        execute : function() {
            var self = this;
            this.initParameters();
            this.showLoading();
            this.setUnDisplayList();
            this.setCssLinkList();
            this.setStyleList();
            this.initExcelElement();
            this.adjustAndRemoveForExcelOut(this.excelElement.head);
            this.adjustAndRemoveForExcelOut(this.excelElement.body);
            this.removeUnDisplayElement();
            this.saveToExcel();
        },
        showLoading : function() {
            //ローディング表示に切り替え
        },
        setUnDisplayList : function() {
            //非表示となっている要素の取得
            
            var self = this;
            
            $("html").find("[style*='none']").each(function() {
                var tag = this.tagName.toLowerCase();

                //hiddenは対象にしない
                if(tag === "input"
                   && $(this).attr("type") === "hidden") {
                    return true;
                }

                //formは対象にしないでおく
                if(tag === "form") {
                    return true;
                }

                //ここブラッシュアップ必要と思われる
                if($(this).css("display") === "none") {
                    var target = {
                        tag : tag,
                        type : null,
                        value : ""
                    };
                    if($(this).attr("id")) {
                        target.type = "id";
                        target.value = $(this).attr("id");
                    }
                    else if($(this).attr("name")) {
                        target.type = "name";
                        target.value = $(this).attr("name");
                    }
                    else if($(this).attr("class")) {
                        target.type = "class";
                        target.value = $(this).attr("class");
                    }

                    //限定できそうな場合のみ追加する
                    //インラインとか、jQuery.cssで直接display:noneにされているとどうしようもないなぁ
                    if(target.type) {
                        self.unDisplayList.push(target);
                    }
                }
            });
            
        },
        spliceUnDisplayList : function(target) {
            //指定されたtargetを取り除く
            
            var tmpList = [];
            for(var i = 0; i < this.unDisplayList.length; i++) {
                if(this.unDisplayList[i].tag === target.tag
                   && this.unDisplayList[i].type === target.type
                   && this.unDisplayList[i].value === target.value) {
                    continue;
                }
                tmpList.push(this.unDisplayList[i]);
            }
            this.unDisplayList = tmpList;
        },
        setCssLinkList : function() {
            //linkタグのうち、外部CSSを利用しているもののURLを取得
            
            var self = this;
            
            $("html").find("link").each(function() {
                
                var tag = this.tagName.toLowerCase();
                
                if(($(this).attr("type") === "text/css" || $(this).attr("rel") === "stylesheet")

                   && $(this).attr("href")) {
                    self.cssLinkList.push($(this).attr("href"));
                }
            });
        },
        setStyleList : function() {
            //styleタグの中身を取得する
            
            var self = this;
            
            $("html").find("style").each(function() {
                
                var tag = this.tagName.toLowerCase();
                
                self.styleList.push($(this).text());
            });
        },
        initExcelElement : function() {
            //EXCELとして保存する為のDOMを初期化

            this.excelElement.html = $("<html>");
            this.excelElement.head = $("<head>");
            
            //head,bodyにhtmlメソッドで渡した時点でスクリプトが内部的に実行されてしまうので、この時点で抜
            //TODO:img、linkタグはどうする？裏でリクエストを投げてしまうかも
            var head = $("head").html().replace(/document\.write\(.+\);/gi, "")
                                       .replace(/<script[^>]+?\/>|<script(.|\s)*?\/script>/gi, "")
                                       .replace(/<noscript>(.|\n|\r\n)+?<\/noscript>/gi, "")
                                       .replace(/<img(.|\s)*?>/gi, "")
                                       .replace(/<link(.|\s)*?>/gi, "");

            var body = $("body").html().replace(/<script[^>]+?\/>|<script(.|\s)*?\/script>/gi, "")
                                       .replace(/<noscript>(.|\n|\r\n)+?<\/noscript>/gi, "")
                                       .replace(/<img(.|\s*)?>/gi, "")
                                       .replace(/<link(.|\s)*?>/gi, "");
            
            this.excelElement.head.html(head);
            this.excelElement.body = $("<body>");
            this.excelElement.body.html(body);
            this.excelElement.html.append(this.excelElement.head);
            this.excelElement.html.append(this.excelElement.body);
        },
        removeUnDisplayElement : function() {
            //EXCEL用DOMから非表示となっている要素の排除
            
            if(this.unDisplayList.length > 0) {
                for(var i = 0; i < this.unDisplayList.length; i++) {
                    
                    var target = this.unDisplayList[i];
                    var tagName =  target.tag.toLowerCase();
                    
                    var selector = "";
                    if(target.type === "id") {
                        selector = tagName + "#" + target.value;
                    }
                    else if(target.type === "name") {
                        selector = tagName + "[name='" + target.value + "']";
                    }
                    else if(target.type === "class") {
                        selector = tagName + "." + target.value;
                    }
                    
                    //ここ動いてる？
                    this.excelElement.html.find(selector).each(function(){
                        this.parentNode.removeChild(this);
                    });
                }
            }
            
        },
        adjustAndRemoveForExcelOut : function(element) {
            //excel用に要素を整形する(jQueryオブジェクトで渡す)
            //excelで出力される内容を修正したい場合にはここに手を入れる

            var tags_for_processing = ['div','tr','th','td','table','input','select','img','script','link','style','button','br','a','textarea'];

            var self = this;
            var showFaceImage = $("#show_face_flag").prop("checked");

            tags_for_processing.forEach(function(tag) {
                $.each(Array.prototype.slice.call(element[0].getElementsByTagName(tag)),function () {
                    try {
                        var $this = $(this);

                        if (!this.tagName) {
                            return true;
                        }

                        var tag = this.tagName.toLowerCase();

                        var text = $.trim($this.text());

                        // 関数が含まれる場合にはテキスト形に変更する。
                        if (self.CONSTS.REG_FORMAT_TEXT.test(text)) {
                            $this.attr("style", "mso-number-format: \\@;");
                        }

                        //まずは新UIUX用の特殊処理を行う
                        self.specificAdjustElementForUIUX201608(this, element);

                        if ((tag === "th" || tag === "td" || tag === "tr") && $this.hasClass("unnecessary_in_excel")) {
                            this.parentNode.removeChild(this);
                            return;
                        }

                        if (tag === "select" && $this.hasClass("range_in_excel")) {
                            var optionValues = [];
                            $this.children("option").each(function (idx, opt) {
                                optionValues.push($(opt).text());
                            });
                            var cell = $this.parent().parent();
                            this.parentNode.parentNode.removeChild(this.parentNode); //パフォーマンスのため、jQueryのremove()を使わない
                            cell.html("<p>" + Math.max.apply(null, optionValues) + " ~ "
                                + Math.min.apply(null, optionValues) + " " + cell.text() + "</p>");
                            return;
                        }

                        if (tag === "img" && $this.hasClass("specific-status_close")) {
                            var alt = $this.attr('alt');
                            $this.replaceWith($("<span>").text(alt));
                            return;
                        }

                        if (tag === "tr" && $this.hasClass("htBlock-vrCalendarTable_weekstart")) {
                            $this.removeClass("htBlock-vrCalendarTable_weekstart");
                            return;
                        }

                        if (tag === "tr" && $this.hasClass("htBlock-rowExpandTable_dummyRow")) {
                            this.parentNode.removeChild(this); //パフォーマンスのため、jQueryのremove()を使わない
                            return;
                        }

                        if (tag === "div" && $this.hasClass("htBlock-rowExpandTable")) {
                            $this.removeClass("htBlock-rowExpandTable");
                            return;
                        }

                        // Excel画像出力
                        if (tag === "th" && showFaceImage && ($this.hasClass("specific_record") || $this.hasClass("employee_name"))) {
                            $this.attr("colspan", 2);
                            self.replaceNumberToString(this);
                            return;
                        }
                        if (tag === "td" && showFaceImage &&
                            ($this.hasClass("start_end_timerecord") || $this.hasClass("rest_timerecord")
                                || $this.hasClass("moving_timerecord") || $this.hasClass("bundle_timerecord")
                                || $this.hasClass("employee_name"))) {
                            var row = $("<td></td>");
                            var images = $this.find("img.face-image").each(function (idx, rowImg) {
                                row.append(rowImg);
                                row.append("<br>");
                            });
                            $this.before(row);
                            $this.find("a").each(function () {
                                this.parentNode.removeChild(this)
                            });
                            /*パフォーマンスのため、jQueryのremove()を使わない */
                            self.replaceNumberToString(this);
                            return;
                        }
                        if (tag === "div" && showFaceImage && $this.hasClass("face_template")) {
                            $this.before($this.find("img.face-image"));
                            $this.find("a").each(function () {
                                this.parentNode.removeChild(this)
                            });
                            /*パフォーマンスのため、jQueryのremove()を使わない */
                            return;
                        }

                        // 日別スケジュール日別業務の時間帯部分の幅調整
                        if (tag === "table" && $this.attr("id") === "time_division_table") {
                            $this.find("td").each(function () {
                                var $this = $(this);
                                if ($.trim($this.text()) === "") {
                                    $this.text(".");
                                    var color = $this.css("background-color");
                                    if (!color) color = "#fff";
                                    $this.css("color", color);
                                    $this.css("text-align", "center");
                                    $this.css("font-size", "9pt");
                                }
                            });
                        }

                        //script,link,imgは一応再検索
                        if (tag === "script"
                            || tag === "link"
                            || tag === "style"
                            || (tag === "img" && !$this.hasClass("face-image"))
                            || tag === "button"
                            || (tag === "br" && $this.hasClass("unnecessary_in_excel"))) {
                            this.parentNode.removeChild(this); //パフォーマンスのため、jQueryのremove()を使わない
                            return true;
                        }

                        if (tag === "a") {
                            $this.replaceWith($("<span>").text($this.text()).css("display", $this.css("display")));
                            return true;
                        }

                        if (tag === "input") {
                            if ($this.attr("type") !== "text" && $this.attr("type") !== "hidden") {
                                this.parentNode.removeChild(this); //パフォーマンスのため、jQueryのremove()を使わない
                                return true;
                            }
                            if ($this.attr("type") === "text") {
                                $this.replaceWith($("<span>").text($this.val()).css("display", $this.css("display")));
                                return;
                            }
                            if ($this.attr("type") === "hidden") {
                                if ($this.hasClass("hidden_working_type_item")
                                    || $this.hasClass("hidden_section_item")
                                    || $this.hasClass("hidden_employee_group_item_0")) {
                                    this.parentNode.removeChild(this);
                                    return;
                                }
                                //他は速度のためregexで消す。
                            }

                            return true;
                        }

                        if (tag === "textarea") {
                            //TODO: divでいい？
                            $this.replaceWith($("<div>").text($this.val()).css("display", $this.css("display")));
                            return true;
                        }

                        if (tag === "select") {
                            $this.replaceWith($("<span>").text($this.find("option:selected").text()).css("display", $this.css("display")));
                            return true;
                        }

                        if (tag === "table") {

                        }

                        if (tag === "div" && $this.hasClass("unnecessary_in_excel")) {
                            var help = $this.children("b");
                            $this.before(help);
                            return;
                        }

                        if (tag === "td"
                            || tag === "th"
                            || tag === "span"
                            || tag === "p") {
                            self.replaceNumberToString(this);
                        }
                    }catch(err){
                        //次の要素にする。
                    }
                })
            });

            if(this.settings.removeTargetSelector && this.settings.removeTargetSelector.length > 0) {
                for(var i = 0, l=this.settings.removeTargetSelector.length; i < l; i++) {

                    var selector = this.settings.removeTargetSelector[i];
                    element.find(selector).each(function() {
                        this.parentNode.removeChild(this); //パフォーマンスのため、jQueryのremove()を使わない
                    });

                }
            }
        },
        replaceNumberToString : function(element) {

            // #00514043
            // 子要素以下にdivタグを含む場合、変換対象としない
            if($(element).find("div").length > 0){
                return;
            }
            if(this.settings.numberToString === "1"){
                var $element = $(element);

                var nonTargetFormat = $element.attr("data-non-target-format");
                if(!nonTargetFormat) {
                    var text = $element.text();
                    if(text === null
                        || text === undefined
                        || text === "") {
                        return;
                    }
                    text = $.trim(text);
                    for(var i = 0,l=this.CONSTS.REGS.length; i < l; i++) {
                        if(this.CONSTS.REGS[i].test(text)) {
                            $element.html($element.html().replace(text, "=t(\"" + text + "\")"));
                            return;
                        }
                    }
                }
            }
        },
        specificAdjustElementForUIUX201608 : function(element, rootElement) {
            //UIUX201608版用の特殊な処理

            var tagName =  element.tagName.toLowerCase();
            var $element = $(element);
            var self = this;
            
            if(tagName === "div") {
                if($element.hasClass("htBlock-tab")
                   || $element.hasClass("htBlock-linedTab")) {
                    $element.children("ul").each(function(){
                        //タブ表示の場合にはタブ部分を削除する
                        this.parentNode.removeChild(this); /*パフォーマンスのため、jQueryのremove()を使わない */
                    });
                }

                //スクロールテーブルはノーマルテーブルに変更してみる
                if($element.hasClass("htBlock-scrollTable")) {
                    $element.removeClass("htBlock-scrollTable")
                            .addClass("htBlock-normalTable");
                }

                //パネル内のコンテンツは表示状態にする
                if($element.hasClass("htBlock-blockPanel_content")) {
                    $element.css("display","");
                    this.spliceUnDisplayList({ tag: "div", type: "class", value: "htBlock-blockPanel_content" });
                }

                if($element.hasClass("htBlock-requestTable")) {
                    $element.find("table").each(function() {
                        this.style.borderColor = "#E6E5E6";
                    });
                }

                /*
                * Excel出力に関係のない specific-table-installationButton クラスを削除する
                * */
                if($element.hasClass("specific-table-installationButton")){
                    $element.removeClass("specific-table-installationButton");
                }

                /*
                * Excelは2つ以上のCSSクラスがある場合は、Parseできないかもしれないので、出力したら、スタイルがなくなってしまう。
                * なので、必要なクラスしか残さない。
                * */
                if($element.hasClass("specific-table") && $element.hasClass("htBlock-normalTable")){
                    $element.removeClass("specific-table");
                }

                if ($element.hasClass("htBlock-autoNewLineTable")) {
                    $element.find("ul").each(function () {

                        //無理やりテーブルに変換して横並びにする
                        var ths = [];
                        var tds = [];

                        $.each(Array.prototype.slice.call(this.getElementsByTagName('li')),function () {
                            var $this = $(this);
                            $this.find("label").each(function () {
                                ths.push($("<th>").text($(this).text()));
                            });
                            $this.find("div").each(function () {
                                // 日数アラートの背景設定
                                var backgroundColor;
                                if ($(this).css("background-color")) {
                                    backgroundColor = $(this).css("background-color");
                                } else {
                                    backgroundColor = "";
                                }
                                tds.push($("<td>").text($(this).text()).css("background-color", backgroundColor))
                            });
                        });

                        /* 以下のhtBlock-normalTableのクラスを利用するため、正しい構造にする */
                        var table = $("<table>");
                        table.append($("<thead>").append($("<tr>").append(ths)))
                            .append($("<tbody>").append($("<tr>").append(tds)));

                        $(this).replaceWith(table);
                    });
                    $element.removeClass().addClass("htBlock-normalTable");
                }

            }
            var setHolidayColor = function (className) {
                if ($element.hasClass(className)) {
                    var color = $("." + className).css("color");
                    if (color !== "") {
                        $element.css("color", color)
                    }
                }
            }
            if (tagName === "th") {
                setHolidayColor("htBlock-hrCalendarTable_saturday");
                setHolidayColor("htBlock-hrCalendarTable_sunday");
                setHolidayColor("htBlock-hrCalendarTableA_saturday");
                setHolidayColor("htBlock-hrCalendarTableA_sunday");
            }
            if (tagName === "td") {
                setHolidayColor("htBlock-scrollTable_sunday");
                setHolidayColor("htBlock-scrollTable_saturday");
                if ($element.hasClass("htBlock-rowExpandTable_foldableRow")) {
                    element.parentNode.parentNode.removeChild(element.parentNode); /*パフォーマンスのため、jQueryのremove()を使わない */
                }

                //#13559 : 退職者の退職日以降のデータがグレーになるようにする。
                if ($element.hasClass("leave")){//退職者
                    //「leave」クラス以外がある場合、削除する。、
                    var classes = $element.attr("class").replace('leave','').trim();
                    if (classes.length > 0){
                        $element.attr("class","");
                        $element.addClass("leave");
                        return;
                    }
                }

                //#9160 スケジュール管理画面　パターン表示色をtdにも適用する
                if ($element.hasClass("htBlock-hrCalendarTable_dayCell")) {
                    (function () {
                        var viewArea = $element.find("p").first();

                        var color, backGroundColor;
                        if (viewArea.length == 1) {
                            color = viewArea.css("color");
                            backGroundColor = viewArea.css("background-color");
                            if (color != "") {
                                $element.css("color", color);
                            }
                            if (backGroundColor != "") {
                                $element.css("background-color", backGroundColor);
                            }
                        }
                    })();
                }

                // スケジュール管理画面　法定休日、法定外休日のボーダー色を変更
                if ($element.hasClass("htBlock-hrCalendarTableA_saturday") || $element.hasClass("htBlock-hrCalendarTableA_sunday")) {

                    (function () {
                        var border = $element.hasClass("htBlock-hrCalendarTableA_saturday") ? "2px solid #5E91EB" : "2px solid #FF645C";

                        // ボーダー色を勤務日種別の色に
                        $element.css("border", border);

                        // 背景色をパターンの色に
                        var viewArea = $element.find("div").first();

                        var color, backGroundColor;
                        if (viewArea.length == 1) {
                            color = viewArea.css("color");
                            backGroundColor = viewArea.css("background-color");
                            if (color != "") {
                                $element.css("color", color);
                            }
                            if (backGroundColor != "") {
                                $element.css("background-color", backGroundColor);
                            }
                        }

                    })();

                }

                if (self.settings.currentPageId === "/schedule/daily_shift_list" && ($element.hasClass("nopad") || $element.hasClass("nopadleft"))) {
                    $element.css("border-color", "black");
                }
            }
            if (tagName === "tr") {
                if ($element.hasClass("htBlock-scrollTable_toolbar")
                    || $element.hasClass("htBlock-rowExpandTable_dummyRow")) {
                    //スクロールテーブルの上部ボタンスペース行を削除する
                    element.parentNode.removeChild(element); /*パフォーマンスのため、jQueryのremove()を使わない */
                }
            }
            if (tagName === "table") {
                if ($element.hasClass("htBlock-scrollTable_fixed")) {
                    //スクロールテーブルの行固定用に作成されたかぶせるためのテーブルを削除
                    element.parentNode.removeChild(element); /*パフォーマンスのため、jQueryのremove()を使わない */
                }

                if ($element.hasClass("htBlock-hrCalendarTableA_fixedHeader")) {
                    element.parentNode.removeChild(element); /*パフォーマンスのため、jQueryのremove()を使わない */
                }

                if ($element.hasClass("htBlock-adjastableTableF_fixedHeader")) {
                    element.parentNode.removeChild(element); /*パフォーマンスのため、jQueryのremove()を使わない */
                }
            }
            if (tagName === "input") {

                var type = $element.attr("type");

                //ラジオボタン、チェックボックスがチェックされていない場合にはそれに紐づくラベルを削除する
                //TODO:条件選択に限定したほうが良い？
                if ((type === "radio"
                        || type === "checkbox")
                    && !$element.prop("checked")) {
                    rootElement.find("label[for=" + $element.attr("id") + "]").each(function () {
                        this.parentNode.removeChild(this); /*パフォーマンスのため、jQueryのremove()を使わない */
                    });
                }

                if (type === "text") {
                    if ($element.hasClass("htBlock-calendar")
                        || $element.hasClass("htBlock-calendarWeek")
                        || $element.hasClass("htBlock-calendarMonth")) {
                        $element.replaceWith($("<span>").text($element.val()).css("display", $element.css("display")));
                    }
                }
            }
        },
        getOutStyleSource : function(index,url) {
            //外部CSSをテキストで取得する
            
            var self = this;
            
            return $.ajax({
                url : url,
                type : "get",
                dataType : "text"
            })
            .done(function(data, textStatus, xhr){
                self.outerStyleList[index] = data;
            });
        },
        saveToExcel : function() {
            var deferreds = [];
            if(this.cssLinkList.length > 0) {
                this.outerStyleList.length = this.cssLinkList.length;
                for(var i = 0,l=this.cssLinkList.length; i<l; i++) {
                    deferreds.push(this.getOutStyleSource(i,this.cssLinkList[i]));
                }
            }
            
            var self = this;
            
            $.when.apply($,deferreds)
            .always(function(){
                
                var styles = self.styleList.concat(self.outerStyleList).join(" ");
                self.excelElement.head.append($("<style>").text(styles));

                var htmlString = self.excelElement.html.html()
                                                       .replace(/<form.*>/g, '')
                                                       .replace(/<\/form.*>/g, '')
                                                       .replace(/<input(.|\s)*?>/gi, "");
                htmlString = "<html>" + htmlString + "</html>";
                self.exportedFile =  new Blob([htmlString], { type: 'application/xls; charset=utf-8' });
                              self.onDownload();
            });
        },
        /*
        * 参照： kingtime.entity.utility.SaveHtmlToExcel.getFileName()
        * Filename = [機能]ページID[yyyyMMddHHmmss].xls
        * */
        getDownloadFileName : function(){
            var appendZero = function(time){
                if (time < 10){
                    return '0' + time;}
                else return '' + time;
            };
                  
            var fileName = this.settings.currentPageId;
            var currentTime = new Date();
            fileName = fileName.replace(/\//, "[");
            fileName = fileName.replace(/\//, "]");
            fileName += '' + currentTime.getFullYear()
                + appendZero(currentTime.getMonth() + 1)
                + appendZero(currentTime.getDate())
                + appendZero(currentTime.getHours())
                + appendZero(currentTime.getMinutes())
                + appendZero(currentTime.getSeconds());
            return fileName + ".xls";
        }
    });
    
    $.fn.KOT_ExcelOut = function(options) {
        this.each(function(){
            new $.KOT_ExcelOut(this, options);
        });
        return this;
    };
})(jQuery);
