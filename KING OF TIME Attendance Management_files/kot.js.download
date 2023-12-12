/*@cc_on
if (@_jscript_version < 9) {
    var _d = document;
    eval("var document = _d");
}
@*/

var clicked=0;
var waitMsec = 4000;
function ckdbclick(waitMsecParam)  {
    if (waitMsecParam != undefined && !isNaN(waitMsecParam)) {
        waitMsec = Number(waitMsecParam);
    }
    setTimeout('timewaitWithButton(\'\', 1, 0, 0)', 0);setTimeout('timewaitWithButton(\'\', 0, 1, 1)', waitMsec);

	if(clicked==1) {
        alert(jsMsgArray["kot_ckdbclick"]);
	    return false;
	}
	else if(clicked==2){
	    clicked = 0;
	    return true;
    }
	else {
	    clicked = 1;
	    return true;
	}
}

function enableDbClick(){
    clicked = 2;
}

var submitButtonArray = new Array();
var timerId;
function timewaitWithButton(buttonId, disableFlag, enableDbClickFlag, clearIntervalFlag) {

    var buttonObj = document.getElementById(buttonId);
    var elementObj;
    var elementStatus = true;

    if (disableFlag == 0) {
        elementStatus = false;
    }
    else {
        elementStatus = true;
    }

    if (buttonObj != null) {
        buttonObj.disabled = elementStatus;
    }

    if (submitButtonArray != null && submitButtonArray.length > 0) {
        for (var i=0; i<submitButtonArray.length; i++) {
            elementObj = document.getElementById(submitButtonArray[i]);
            if (elementObj != null) {
                elementObj.disabled = elementStatus;
            }
        }
    }

    if (enableDbClickFlag == 1) {
        enableDbClick();
    }

    if (clearIntervalFlag == 1) {
        clearInterval(this.timerId);
    }
}

//全体の名前空間
var KOT_GLOBAL = {};
(function(){
    
    //ユーティリティ用名前空間 
    //kot.jsに新しく何かを追加する場合は基本この中にお願いします
    KOT_GLOBAL.KOT_LIB = {};

    // 多重submitを防止する
    // option ページによって挙動を変えるためのオプション指定 (基本は未指定。データ出力系の画面等で挙動を変えたい場合に指定。)
    // {
    //    debug: trueの場合、コンソールにログを表示する
    //    showAlert: trueの場合、2度押し防止メッセージを表示する
    //    duration: ミリ秒指定がある場合にはその時間を超過すれば再submit可能とする
    //    exclusionDurationCheckTarget : [ {id:上記の間隔チェック対象外とするsubmitterのID, message: 独自に表示するメッセージ。指定が無ければデフォルトのメッセージを表示} ]
    // }
    KOT_GLOBAL.KOT_LIB.blockDbClick = function(option) {
        // submit済みフラグ
        var submitted = false;
        // 前回submit時刻
        var prevSubmitTime = null;
        // 1度しか行なえない操作の制御
        var submittedExclusions = {};
        window.addEventListener("submit", function(e) {
            // submit元の要素
            var submitter = e.submitter;
            if (submitted) {

                // 2度押し防止するかどうか
                var isBlock = true;
                // 表示するメッセージのデフォルト値
                var message = jsMsgArray["kot_ckdbclick"]; // ボタンは一度だけ押して下さい

                // 再submit可能な間隔の指定がある場合
                var duration = option.duration;
                if (duration) {
                    // データ出力などは1度しか行えないようにしたいため、再submit可能な間隔が指定されていても再submit対象外かどうかチェックする
                    var exclusions = option.exclusionDurationCheckTarget ? option.exclusionDurationCheckTarget : [];

                    // 再submit対象外とするか
                    var exclusionTarget = false;
                    for (var i=0; i< exclusions.length; i++) {
                        var target = exclusions[i];
                        if (submitter && submitter.id === target.id) {
                            if (!submittedExclusions[target.id]) {
                                isBlock = false;
                            }
                            exclusionTarget = true;
                            if (target.message) {
                                message = target.message;
                            }
                            break;
                        }
                    }

                    // 前回submit時間との差分をチェック
                    var diff = new Date() - prevSubmitTime;
                    if (!exclusionTarget && diff >= duration) {
                        isBlock = false;
                    }
                }

                // 2度押し防止制御
                if (isBlock) {
                    // メッセージ表示を行う場合にはメッセージを表示する
                    if (option.showAlert) {
                        alert(message);
                    }

                    if (option.debug) {
                        console.log("isBlock");
                        console.log(submitter);
                    }

                    e.stopPropagation();
                    e.preventDefault();
                    return false;
                }
            }
            submitted = true;
            submittedExclusions[submitter.id] = true;
            prevSubmitTime = new Date();
        });
        // ブラウザバックした際に操作できなくなることを防ぐ
        // ブラウザバックで戻った場合には送信済みフラグを元に戻す
        window.addEventListener("pageshow", function(e) {
            if (!submitted) return;

            if (e.persisted) {
                submitted = false;
                return;
            }
            var entries = performance.getEntriesByType("navigation");
            entries.forEach(function(entry) {
                if (entry.type === "back_forward") {
                    submitted = false;
                    return;
                }
            });
        });
    };
    
    //オブジェクトをJSON文字列に変換
    //かなり簡易。
    KOT_GLOBAL.KOT_LIB.stringify = function(obj) {
        var type = typeof(obj);
        if(type != "object" || obj == null) {
            if(type == "string") {
                obj = '"' + obj.replace(/\"/g,"\\\"") + '"';
            }
            return String(obj);
        }
        else {
            var key,val,data = [],isArray = (obj && obj.constractor == Array);
            for(key in obj) {
                if(obj.hasOwnProperty(key)) {
                    val = obj[key];
                    type = typeof(val);
                    
                    if(type == "string") {
                        val = '"' + val.replace(/"/g,"\\\"") + '"';
                    }
                    else if(type == "object" && val != null){
                        val = KOT_GLOBAL.KOT_LIB.stringify(val);
                    }
                    data.push((isArray ? "" : '"' + key + '":') + String(val));
                }
            }
            return (isArray ? "[" : "{") + String(data) + (isArray ? "]" : "}");
        }
    };

    // string.formatの代わり
    // 例） KOT_GLOBAL.KOT_LIB.stringFormat("hello {param1} world.{param2}", {param1 : 123456, param2 : "hello"});
    KOT_GLOBAL.KOT_LIB.stringFormat = function(message, obj) {
        return message.replace(/\{(\w+)\}/g, function (m, k) {
            return obj[k];
        });
    };

    KOT_GLOBAL.KOT_LIB.getRootURL = function(scriptFileName) {
        var rootUrl;
        var scripts = document.getElementsByTagName("script");
        var i = scripts.length;
        // /js/(subDirName/の0回以上の繰り返し) + scriptFileName + ("?" + 1回以上の数字の繰り返し)が0回以上繰り返して末尾
        var reg = '(^|.*\/)js\/(.*\/)*' + scriptFileName + '(\\?[0-9]+)*$';
        while (i--) {
            var match = scripts[i].src.match(reg);
            if (match) {
                rootUrl = match[1];
                break;
            }
        }
        return rootUrl;
    };
    
    KOT_GLOBAL.KOT_LIB.makeHiddenByFormId = function(hiddenName, hiddenId, hiddenValue, formId) {
        
        var form = $("#" + formId);
        
        //対象のフォームが無い場合は何もしない
        
        if(!$(form).attr("id")) {
            return;
        }
        if($(form).prop("tagName") !== "FORM") {
            return;
        }
        
        var hidden = $("#" + hiddenId);
        if($(hidden).attr("id")
           && $(hidden).prop("tagName") === "INPUT"
           && $(hidden).attr("type") === "hidden") {
            //同じIDのhiddenが存在する場合は値のみ上書き
            //指定したフォームの下にある保証ないけど
            $(hidden).val(hiddenValue);
        }
        else {
            hidden = $("<INPUT>").attr("type", "hidden").attr("name", hiddenName).val(hiddenValue);
            if(hiddenId) {
                hidden.attr("id", hiddenId);
            }
            $(form).append($(hidden));
        }
    };
    
    KOT_GLOBAL.KOT_LIB.checkGooutFlagTogether = function() {
        var summarySectionType = document.getElementById("summary_section_type");
        var goOutFlag = document.getElementById("goout_flag");
        if(goOutFlag != undefined && goOutFlag != null && goOutFlag.tagName == "INPUT" && goOutFlag.type == "checkbox") {
           if(summarySectionType.checked) {
               goOutFlag.checked = true;
           }
           else {
               goOutFlag.checked = false;
           }
        }
    };
    
    //指定したIdの箇所に指定したIdのCloneを追加する
    KOT_GLOBAL.KOT_LIB.appendChildTargetClone = function(cloneTargetId,addTargetId){
        var cloneTarget = document.getElementById(cloneTargetId);
        var input = cloneTarget.cloneNode(true);
        input.id = cloneTargetId + "_new";
        
        var addTarget = document.getElementById(addTargetId);
        addTarget.appendChild(input);
        return input;
    };

    KOT_GLOBAL.KOT_LIB.decodeEscapeWhiteSpace = function(s) {
        if(s) {
            //末尾のセミコロンが抜けている場合があるので一応両方行う
            return s.replace(/&nbsp;/g," ").replace(/&nbsp/g," ");
        }
        return s;
    };
    
    KOT_GLOBAL.KOT_LIB.isArray = function(array) {
        return Object.prototype.toString.call(array) === "[object Array]";
    };
    
    KOT_GLOBAL.KOT_LIB.makeCsvStringFromArray = function(array) {
        
        var str = "";
        if(!array || !KOT_GLOBAL.KOT_LIB.isArray(array)) {
            return str;
        }
        
        if(array.length == 0) {
            return str;
        }
        
        for(var i = 0,l = array.length; i < l; i++) {
            str += array[i];
            if(i < array.length -1) {
                str += ",";
            }
        }
        return str;
    };

    // 日、時、分をもとにdateオブジェクトを返します
    KOT_GLOBAL.KOT_LIB.toTime = function(day,hour,minute) {
        if (isNaN(day) || isNaN(hour) || isNaN(minute)) {
          return null;
        }
        var date = null;
        try {
          date = new Date(2012,9,1,hour,minute,0);
          date.setTime(date.getTime() + 60 * 60 * 24 * 1000 * day);
        } catch (e) {
          alert(e);
        }
        return date;
    };
    
    // 時間を検証します
    KOT_GLOBAL.KOT_LIB.isValidHour = function(hour, overLimitHour) {
        if (hour == "") return true;
        hour = parseInt(hour, 10);
        if (isNaN(hour)) {
          return false;
        }
        
        
        overLimitHour = !overLimitHour ? 0 : overLimitHour;
        
        if (hour < 0 || 23+overLimitHour < hour) {
          return false;
        }
        return true;
    };
    
    // 分を検証します
    KOT_GLOBAL.KOT_LIB.isValidTime = function(minute) {
        if (minute == "") return true;
        minute = parseInt(minute, 10);
        if (isNaN(minute)) {
          return false;
        }
        if (minute < 0 || 59 < minute) {
          return false;
        }
        return true;
    };
    
    //ブラウザ種類判定
    KOT_GLOBAL.KOT_LIB.getBrowserType = function() {
        var browserType=0;
        var userAgent  = window.navigator.userAgent.toLowerCase();
        var appVersion = window.navigator.appVersion.toLowerCase();
        
        if (userAgent.indexOf("msie") > -1) {
          if (appVersion.indexOf("msie 6.0") > -1) {
            browserType = 106;
          }
          else if (appVersion.indexOf("msie 7.0") > -1) {
            browserType = 107;
          }
          else if (appVersion.indexOf("msie 8.0") > -1) {
            browserType = 108;
          }
          else if (appVersion.indexOf("msie 9.0") > -1) {
            browserType = 109;
          } else if (appVersion.indexOf("msie 10.0") > -1) {
            browserType = 110;
          }
          else {
            browserType = 100;
          }
        }else if (userAgent.indexOf('trident/7') > -1) {
            //IE11
            browserType = 111;
        }
        else if (userAgent.indexOf("firefox") > -1) {
          browserType = 201;
        }
        else if (userAgent.indexOf("opera") > -1) {
          browserType = 301;
        }
        else if (userAgent.indexOf("chrome") > -1) {
          browserType = 401;
        }
        else if (userAgent.indexOf("safari") > -1) {
          browserType = 501;
        }
        else {
          browserType = 999;
        }
        
        return browserType;
    };
    
    //スケジュール汎用フラグDisabledクリア
    KOT_GLOBAL.KOT_LIB.scheduleMultiPurposeFlagClearDisabled = function(selectedSchedulePattern, targetCheckBox1, targetCheckBox2) {
        if (targetCheckBox1 != null && targetCheckBox1.disabled) {
            if (selectedSchedulePattern == null) {
                targetCheckBox1.disabled = false;
                targetCheckBox1.checked = false;
            }
            else {
                if (selectedSchedulePattern.scheduleMultiPurposeFlag1 == null || selectedSchedulePattern.scheduleMultiPurposeFlag1 == "") {
                  targetCheckBox1.disabled = false;
                  targetCheckBox1.checked = false;
                }
            }
        }
        
        if (targetCheckBox2 != null && targetCheckBox2.disabled) {
            if (selectedSchedulePattern == null) {
                targetCheckBox2.disabled = false;
                targetCheckBox2.checked = false;
            }
            else {
                if (selectedSchedulePattern.scheduleMultiPurposeFlag2 == null || selectedSchedulePattern.scheduleMultiPurposeFlag2 == "") {
                    targetCheckBox2.disabled = false;
                    targetCheckBox2.checked = false;
                }
            }
        }
    };
    
    // スケジュール汎用フラグの強制設定
    KOT_GLOBAL.KOT_LIB.scheduleMultiPurposeFlagForceSetting = function(selectedSchedulePattern, targetCheckBox1, targetCheckBox2) {
        
        KOT_GLOBAL.KOT_LIB.scheduleMultiPurposeFlagClearDisabled(selectedSchedulePattern, targetCheckBox1, targetCheckBox2);
        
        if (selectedSchedulePattern != null) {
          if (targetCheckBox1 != null) {
            if (selectedSchedulePattern.scheduleMultiPurposeFlag1 != null && selectedSchedulePattern.scheduleMultiPurposeFlag1 != "") {
              if (selectedSchedulePattern.scheduleMultiPurposeFlag1 == "1") {
                targetCheckBox1.checked = true;
              }
              else {
                targetCheckBox1.checked = false;
              }
              targetCheckBox1.disabled = true;
            }
          }
          
          if (targetCheckBox2 != null) {
            if (selectedSchedulePattern.scheduleMultiPurposeFlag2 != null && selectedSchedulePattern.scheduleMultiPurposeFlag2 != "") {
              if (selectedSchedulePattern.scheduleMultiPurposeFlag2 == "1") {
                targetCheckBox2.checked = true;
              }
              else {
                targetCheckBox2.checked = false;
              }
              targetCheckBox2.disabled = true;
            }
          }
        }
    };

    // スケジュール汎用フラグの強制オフ
    KOT_GLOBAL.KOT_LIB.scheduleMultiPurposeFlagForceDisabled = function(targetCheckBox1, targetCheckBox2) {
        if (targetCheckBox1 != null) {
            targetCheckBox1.checked = false;
            targetCheckBox1.disabled = true;
        }
        if (targetCheckBox2 != null) {
            targetCheckBox2.checked = false;
            targetCheckBox2.disabled = true;
        }
    };

    /*全画面でモーダルウィンドウメッセージ表示
    *  
    * sizesオブジェクトが必ず必要
    * 絶対値指定
    * sizes : {
    *     window : {
    *         width : window幅
    *         heigth : window高さ
    *     },
    *     img : {
    *         width : イメージ幅
    *         heigth : イメージ高さ
    *         top : イメージ縦位置
    *         left : イメージ横位置
    *     },
    *     message : {
    *         fontSize : メッセージフォントサイズ
    *         top : メッセージ縦位置
    *         left : メッセージ横位置
    *     }
    * }
    *
    */
    KOT_GLOBAL.KOT_LIB.showMessageModal = function(imagePath, message, sizes) {
        
        /*
         * imagePath : "images/unprocessedalert/loading.gif"
         * message : "しばらくお待ちください・・・"
         * 
         * 上記の内容に合わせたサイズをデフォルトとする
         */
        var targetSizes = {
                window : { width : 250, height : 60 },
                img : { width : 36, height : 36, left : 12, top : 12 },
                message : { fontSize : 16, left : 60, top : 23}
        };
        
        
        message = message ? message : jsMsgArray["loading_modal_message_default"];
        
        sizes = sizes ? sizes : {};
        jQuery.extend(targetSizes, sizes); //targetSizesにsizesをマージ
        
        //Window部分
        var modalWindow = jQuery("<div>")
                                 .addClass("modal-message-window")
                                 .css("width", targetSizes.window.width)
                                 .css("height", targetSizes.window.height);
        if(imagePath) {
            modalWindow.append(jQuery("<img>")
                                      .addClass("modal-message-image")
                                      .css("width", targetSizes.img.width)
                                      .css("height", targetSizes.img.height)
                                      .css("left", targetSizes.img.left)
                                      .css("top", targetSizes.img.top)
                                      .attr("src",imagePath));
        }
        modalWindow.append(jQuery("<div>")
                                  .addClass("modal-message-message")
                                  .css("font-size", targetSizes.message.fontSize + "px")
                                  .css("left", targetSizes.message.left)
                                  .css("top", targetSizes.message.top)
                                  .text(message));
        
        //大枠
        var wrapper = jQuery("<div>")
                             .addClass("modal-message-wrapper")
                             .append(jQuery("<div>").addClass("modal-message-back"))
                             .append(modalWindow);
        
        jQuery(document.body).append(wrapper);
        
        //表示位置調整
        modalWindow.css("top", (wrapper.height() / 2) - (targetSizes.window.height / 2)).css("left", (wrapper.width() / 2) - (targetSizes.window.width / 2));
        modalWindow.show();
        
        jQuery(window).resize(function(){
            modalWindow.css("top", (wrapper.height() / 2) - (targetSizes.window.height / 2)).css("left", (wrapper.width() / 2) - (targetSizes.window.width / 2));
        });
    };

    KOT_GLOBAL.KOT_LIB.summarySectionTypeClicked = function(targetObject) {
        var substitudeHolidayMinusObject = document.getElementById("substitude_holiday_minus_flag");
        var exportSummaryGooutSectionType = document.getElementById("export_summary_goout_section_type");
        if (typeof targetObject != "undefined" &&
            targetObject.type == "checkbox" &&
            targetObject != null) {
            if(substitudeHolidayMinusObject != null) {
                if (targetObject.checked == true) {
                    substitudeHolidayMinusObject.checked = false;
                }
            }
            if (exportSummaryGooutSectionType != null) {
                if (targetObject.checked == true) {
                    exportSummaryGooutSectionType.checked = false;
                }
            }
        }
    };

    KOT_GLOBAL.KOT_LIB.openWindowForPrint = function( actionTarget, pageId ) {

        if (actionTarget == null) {
            actionTarget = "/admin";
        }

        var printWindow = window.open(actionTarget + '?page_id='+ pageId + '&r=' + Math.random() , "print" ,"width=1200,height=768,toolbar=yes,resizable=yes,location=no,menubar=yes,status=yes,scrollbars=yes");
        printWindow.focus();
    };

    KOT_GLOBAL.KOT_LIB.openHelpNew = function(link) {
        var helpWindow = window.open(link,'help','width=1000,height=800,toolbar=no,resizable=yes,scrollbars=yes,statusbar=yes');
        helpWindow.focus();
    };

    KOT_GLOBAL.KOT_LIB.enableNumeric = function(obj) {
        if (window.event) {
            var c = window.event.keyCode;
            //numericKey = 48-57 & 96-105 / leftKey = 37 / rightKey = 39 / BS = 8
            //10Key0 = 45 / 10Key. = 46 / Home = 36 / End = 35 / Tab = 9 / Shift = 16
            if ( 48<=c&&c<=57 || 96<=c&&c<=105 || c==37 || c==39 || c==8 || c==45 || c==46 || c==36 || c==35 || c==9 || c==16) {
                return;
            }
        }
        var v = obj.value;
        v = v.replace(/[^0-9]/g,'');
        //v=v.replace(/[^0-9!"#$%&'\(\)]/g,'');
        obj.value=v;
    };

    KOT_GLOBAL.KOT_LIB.enableNumericAndComma = function(obj) {
        if (window.event) {
            var c = window.event.keyCode;
            //numericKey = 48-57 & 96-105 / leftKey = 37 / rightKey = 39 / BS = 8 / dot = 190
            //10Key0 = 45 / 10Key. = 46 / Home = 36 / End = 35 / Tab = 9 / Shift = 16 dot = 110
            if ( 48<=c&&c<=57 || 96<=c&&c<=105 || c==37 || c==39 || c==8 || c==45 || c==46 || c==36 || c==35 || c==9 || c==16 || c==110 || c==190) {
                return;
            }
        }
        var v = obj.value;
        v = v.replace(/[^0-9+\.]/g,'');
        obj.value=v;
    };

    KOT_GLOBAL.KOT_LIB.getElementsByClassNameAndTagNameFromObject = function(obj, className, tagName) {
        var i, j, eltClass;
        var objCN = [];
        if (obj == null) {
            return objCN;
        }
        var objAll = obj.getElementsByTagName(tagName);
        for (i = 0; i < objAll.length; i++) {
            eltClass = objAll[i].className.split(/\s+/);
            for (j = 0; j < eltClass.length; j++) {
                if (eltClass[j] == className) {
                    objCN.push(objAll[i]);
                    break;
                }
            }
        }
        return objCN;
    };

    //indexOfの代替(動作仕様的にはcontains()と同義)
    KOT_GLOBAL.KOT_LIB.checkIndex = function(array,obj) {
        if(array == null || array.length == 0) {
            return false;
        }
        else {
            for(var i = 0; i < array.length; i++) {
                if(array[i] == obj) {
                    return true;
                }
            }
            return false;
        }
    };

    KOT_GLOBAL.KOT_LIB.goHistory = function(value){
        history.go(value);
    };

    KOT_GLOBAL.KOT_LIB.escapeHTML = function(s) {
        if(s) {
            return s.replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/\"/g,"&quot;").replace(/\'/g,"&#39;").replace(/\\/g, "&#92;");
        }
    };

    KOT_GLOBAL.KOT_LIB.decodeEscapeHTML = function(s) {
        if(s) {
            return s.replace(/&amp;/g,"&").replace(/&lt;/g,"<").replace(/&gt;/g,">").replace(/&quot;/g,"\"").replace(/&#39;/g,"\'").replace(/&#92;/g,"\\");
        }
    };

    KOT_GLOBAL.KOT_LIB.openWindowWithoutNavigation = function( URL, width, height ) {
        if (width == null) {
            width = 1000;
        }
        if (height == null) {
            height = 800;
        }
        window.open(URL,'id','width=' + width + ',height=' + height + ',toolbar=no,resizable=yes,scrollbars=yes,statusbar=yes');
    };

    KOT_GLOBAL.KOT_LIB.makeHidden = function(name,value) {
        var elem = document.getElementById(name);
        if(elem){
            elem.value = value;
        }
        else{
            var q = document.createElement('input');
            q.type = 'hidden';
            q.name = name;
            q.id   = name;
            q.value = value;

            document.forms[0].appendChild(q);
        }
    };

    KOT_GLOBAL.KOT_LIB.makeHiddenAndSetFormNo = function(name,value,formno) {
        var elem = document.getElementById(name);
        if(elem){
            elem.value = value;
        }
        else{
            var q = document.createElement('input');
            q.type = 'hidden';
            q.name = name;
            q.id   = name;
            q.value = value;

            document.forms[formno].appendChild(q);
        }
    };

    KOT_GLOBAL.KOT_LIB.changeDateSelectionType = function(nextDateSelectionType) {
        KOT_GLOBAL.KOT_LIB.makeHidden("next_date_selection_type", nextDateSelectionType);
    };

    KOT_GLOBAL.KOT_LIB.onClickActionButton = function(actionInputId) {
        jQuery("#" + actionInputId).click();
    };

    // SSOログインを確認して、SSOログインされている場合解除の内容をユーザに知らせる
    KOT_GLOBAL.KOT_LIB.checkConfirmSso = function(ssoLogin) {
        if (ssoLogin) {
            return confirm(jsMsgArray["sso_confirm_message"]);
        } else {
            return true;
        }
    };

    KOT_GLOBAL.KOT_LIB.NumericValue={};
    KOT_GLOBAL.KOT_LIB.NumericValue.processNumericContent = function (event, obj, cursorPosition, previousVal) {
        var character = String.fromCharCode(event.which);
        if ('-0123456789０１２３４５６７８９.'.indexOf(character, 0) < 0) {
            //stop key event firing for non numeric characters.
            return false;
        }else {
            var c = window.event.keyCode;
            if(c==45 && (previousVal.indexOf('-',0) < 0) && cursorPosition == 0){ //minus sign validation. (allow in front of number only)
                return true;
            }else if(c==46 && (previousVal.indexOf('.',0) < 0)){ // dot sign validation. (allow only one dot)
                return true;
            }else if(c != 45 && c != 46){
                return true;
            }
        }
    };

    KOT_GLOBAL.KOT_LIB.NumericValue.validateNumericValue = function (elementId) {
        var previousVal;
        var textBoxElement = document.getElementById(elementId);
        var jqueryId = "#"+elementId;
        $(jqueryId).keypress(function(e) {
            previousVal = textBoxElement.value;
            var cursorPosition = textBoxElement.selectionStart;
            if(KOT_GLOBAL.KOT_LIB.NumericValue.processNumericContent(e, textBoxElement, cursorPosition, previousVal)){
                return true;
            }else{
                return false;
            }
        });
    };

    KOT_GLOBAL.KOT_LIB.TimeValue = {};
    KOT_GLOBAL.KOT_LIB.TimeValue.REGS = {
        timePattern : /^[0-9]*[0-9]+:?[0-5]*[0-9]$/,
        hourPattern : /^[0-9]*[0-9]+$/,  // 25時のような入力を許容させるため
        minutePattern : /^[0-5]*[0-9]+$/
    };
    // overLimitHourは23時を超えて何時間入力を許容させるか(25時のような入力を行うため)
    KOT_GLOBAL.KOT_LIB.TimeValue.timeToHourMinute = function(time, overLimitHour) {
        
        var returnObj = {
            hour : "",
            minute : ""
        };
        
        //3桁以上の入力を許容
        if(!time || time.length < 3 || time.length > 5) {
            return returnObj;
        }
        time = time.replace(/[０-９ ：]/g, function (c) {
            return String.fromCharCode(c.charCodeAt(0) - 0xFEE0);
        });
        if(!KOT_GLOBAL.KOT_LIB.TimeValue.REGS.timePattern.test(time)) {
            return returnObj;
        }
        
        var hourMinuteArray;
        if(time.indexOf(":") != -1) {
            hourMinuteArray = time.split(":");
        }
        else {
            hourMinuteArray = [];
            if(time.length == 3) {
              hourMinuteArray[0] = time.substring(0, 1);
              hourMinuteArray[1] = time.substring(1);
            }
            else {
              hourMinuteArray[0] = time.substring(0, 2);
              hourMinuteArray[1] = time.substring(2);
            }
        }
        
        if(KOT_GLOBAL.KOT_LIB.isValidHour(hourMinuteArray[0], overLimitHour) && KOT_GLOBAL.KOT_LIB.isValidTime(hourMinuteArray[1])) {
            if(hourMinuteArray[0].length == 1) {
                returnObj.hour = "" + "0" + hourMinuteArray[0];
            }
            else {
                returnObj.hour = hourMinuteArray[0];
            }
            
            if(hourMinuteArray[1].length == 1) {
                returnObj.minute = "" + "0" + hourMinuteArray[1];
            }
            else {
                returnObj.minute = hourMinuteArray[1];
            }
        }
        
        return returnObj;
    };
    KOT_GLOBAL.KOT_LIB.TimeValue.hourMinuteToTime = function(hour, minute) {
        
        var returnObj = {
            time : ""
        };
        
        if(!KOT_GLOBAL.KOT_LIB.TimeValue.REGS.hourPattern.test(hour)
           || !KOT_GLOBAL.KOT_LIB.TimeValue.REGS.minutePattern.test(minute)) {
            return returnObj;
        }
        
        if(!KOT_GLOBAL.KOT_LIB.isValidHour(hour) || !KOT_GLOBAL.KOT_LIB.isValidTime(minute)) {
            return returnObj;
        }
        
        if(hour.length == 1) {
            hour = "" + "0" + hour;
        }
        
        if(minute.length == 1) {
            minute = "" + "0" + minute;
        }
        
        returnObj.time = hour + ":" + minute; //とりあえず必ず":"区切りにしてみる
        
        return returnObj;
    };
    
    KOT_GLOBAL.KOT_LIB.DateValue = {};
    KOT_GLOBAL.KOT_LIB.DateValue.REGS = {
        datePattern : [/^[0-9]{4}[0-1]{1}[0-9]{1}[0-3]{1}[0-9]{2}$/, /^[0-9]{4}\/+[0-1]?[0-9]{1}\/+[0-3]?[0-9]{1}$/],
        yearPattern : /^[0-9]{4}$/,
        monthPattern : /^[0-1]?[0-9]{1}$/,
        dayPattern : /^[0-3]?[0-9]{1}$/
    };
    KOT_GLOBAL.KOT_LIB.DateValue.dateToYearMonthDay = function(date) {
        
        var returnObj = {
            year : "",
            month : "",
            day : ""
        };
        
        if(!date 
           || (!KOT_GLOBAL.KOT_LIB.DateValue.REGS.datePattern[0].test(date) && !KOT_GLOBAL.KOT_LIB.DateValue.REGS.datePattern[1])) {
            return returnObj;
        }
        
        var dateArray;
        if(KOT_GLOBAL.KOT_LIB.DateValue.REGS.datePattern[1].test(date)) {
            // yyyy/mm/dd
            dateArray = date.split("/");
        }
        else {
            dateArray = [];
            dateArray[0] = date.substring(0,3);
            dateArray[1] = date.substring(4,5);
            dateArray[2] = date.substring(6);
        }
        
        returnObj.year = dateArray[0];
        returnObj.month = dateArray[1];
        returnObj.day = dateArray[2];
        
        return returnObj;
    };
    
    /**** 半休スケジュール判定用の部品群 ****/
    KOT_GLOBAL.KOT_LIB.HalfDayType = {};
    
    //Get leave type half day schedule pattern
    KOT_GLOBAL.KOT_LIB.HalfDayType.getHalfDayLeaveTypeSchedulePattern = function(patternObj, halfDayTypeCode) {

        var tmpPattern = jQuery.extend(true, {}, patternObj);
        var tmpHalfDayPattern;
        if (patternObj && halfDayTypeCode) {
            if (halfDayTypeCode === "2") {
                if (patternObj.pmPattern) {
                    tmpHalfDayPattern = patternObj.pmPattern;
                }
            } else if (halfDayTypeCode === "3") {
                if (patternObj.amPattern) {
                    tmpHalfDayPattern = patternObj.amPattern;
                }
            }
        }

        if (tmpHalfDayPattern) {
            var tmpHalfDayBreakTimeArray = tmpHalfDayPattern.breakTimeArray;
            tmpPattern = jQuery.extend(true, tmpPattern, tmpHalfDayPattern);
            tmpPattern.breakTimeArray = jQuery.extend(true, [], tmpHalfDayBreakTimeArray);
        }

        if (!tmpPattern.schedulePatternName) {
            tmpPattern = undefined;
        }

        return tmpPattern;
    };
    
    //半休のパターンObjを返す
    KOT_GLOBAL.KOT_LIB.HalfDayType.getHalfDaySchedulePattern = function(patternObj, halfDayTypeCode) {
        
        
        var tmpPattern = jQuery.extend(true, {}, patternObj);
        
        var tmpHalfDayPattern;
        if (patternObj && halfDayTypeCode) {
          //半休が設定されている場合には半休スケジュールを適用
          var halfDayCheck = KOT_GLOBAL.KOT_LIB.HalfDayType.checkHalfDaySelectType(halfDayTypeCode);
          if(halfDayCheck === "1") {
            if(patternObj.pmPattern) {
                tmpHalfDayPattern = patternObj.pmPattern;
            }
          }
          else if(halfDayCheck === "2") {
            if (patternObj.amPattern) {
                tmpHalfDayPattern = patternObj.amPattern;
            }
          }
        }
        
        if(tmpHalfDayPattern) {
            var tmpHalfDayBreakTimeArray = tmpHalfDayPattern.breakTimeArray;
            tmpPattern = jQuery.extend(true, tmpPattern, tmpHalfDayPattern);
            tmpPattern.breakTimeArray = jQuery.extend(true, [], tmpHalfDayBreakTimeArray);
        }
        
        if(!tmpPattern.schedulePatternName) {
            tmpPattern = undefined;
        }
        
        return tmpPattern;
    };
    
    //今どの半休を選択しているか
    KOT_GLOBAL.KOT_LIB.HalfDayType.checkHalfDaySelectType = function(halfDayTypeCode) {
        //文字列にしてる
        halfDayTypeCode = "" + halfDayTypeCode;
        
        if(halfDayTypeCode.indexOf("1",0) == 0) {
            return "1"; //AM
        }
        if(halfDayTypeCode.indexOf("2",0) == 0) {
            return "2"; //PM
        }
        return "";
    };

    //Get half day leave type code
    KOT_GLOBAL.KOT_LIB.HalfDayType.checkHalfDayLeaveSelectType = function(halfDayTypeCode) {

            if(halfDayTypeCode == 2) {
                return "1"; //AM
            }
            if(halfDayTypeCode == 3) {
                return "2"; //PM
            }
            return "";
    };
    
    //パターンを選んだ際に設定されている半休のみ選択可能にする
    KOT_GLOBAL.KOT_LIB.HalfDayType.checkOneDaySelectableHalfDaySchedule = function(scheduleTypeCodeArray,selectedScheduleTypeCode,halfDaySelectableCondtidion) {
        
        var halfDayTypeMultiplyAmount = 10000000; //スケージュールパターンの方の乗数を100万に変更されたため、合わせる。
        var selectedHalfDayTypeStatus = parseInt(selectedScheduleTypeCode.value / halfDayTypeMultiplyAmount);
        var browserType = KOT_GLOBAL.KOT_LIB.getBrowserType();
        var isClassicMode = false;
        if (browserType == 106 || browserType == 107) {
            isClassicMode = true;
        }
        
        for(var i = 0; i < scheduleTypeCodeArray.length; i++) {
            var scheduleTypeCode = scheduleTypeCodeArray.options[i];
            if(scheduleTypeCode != null && scheduleTypeCode.value != null && scheduleTypeCode.value != "") {
                //半休種別(0=通常 / 1=AM半休 / 2=PM半休)
                var halfDayTypeStatus = parseInt(scheduleTypeCode.value / halfDayTypeMultiplyAmount);
                
                var isDisable = false;
                
                //半休設定不可
                if((halfDaySelectableCondtidion == null || halfDaySelectableCondtidion == 0) &&
                    halfDayTypeStatus != 0 ) {
                    isDisable = true;
                }
                //PM半休設定あり
                else if(halfDaySelectableCondtidion == 1 && halfDayTypeStatus == 1) {
                    isDisable = true;
                }
                //AM半休設定あり
                else if(halfDaySelectableCondtidion == 2 && halfDayTypeStatus == 2) {
                    isDisable = true;
                }
                
                if (isDisable) {
                    scheduleTypeCode.disabled = true;
                    if(isClassicMode) {
                        scheduleTypeCode.style.color = "graytext";
                    }
                    if (halfDayTypeStatus == 0 || halfDayTypeStatus == selectedHalfDayTypeStatus) {
                        scheduleTypeCodeArray[0].selected = true;
                    }
                }
                else {
                    scheduleTypeCode.disabled = false;
                    if(isClassicMode) {
                        scheduleTypeCode.style.color = "#000000";
                    }
                }
            }
        }
    };

    //休暇取得が新方式
    KOT_GLOBAL.KOT_LIB.HalfDayType.checkOneDaySelectableHalfDayScheduleNew = function(selectedSchedulePatternId,halfDaySelectableCondtidion,leave_type_selection){
        var setAMLeaveDisable = true;
        var setPMLeaveDisable = true;
        var selectedOptions = leave_type_selection.val();// 選択した取得単位

        var halfDayLeaveDisabledPatternIsSelected = selectedSchedulePatternId == null
                                                || selectedSchedulePatternId === ''
                                                || halfDaySelectableCondtidion == null
                                                || halfDaySelectableCondtidion === 0;
        var AMLeaveDisablePatternIsSelected = halfDaySelectableCondtidion === 1;
        var PMLeaveDisablePatternIsSelected = halfDaySelectableCondtidion === 2;
        var AMLeaveIsSelected = (selectedOptions == 2);
        var PMLeaveIsSelected = (selectedOptions == 3);

        if (halfDayLeaveDisabledPatternIsSelected){// 「ーー」または半日勤務設定のないパターンを選択した
            if ( AMLeaveIsSelected || PMLeaveIsSelected){ // PM半休
                // 選択したものをクリア
                leave_type_selection.find('option:first').prop('selected', true);
            }
            setAMLeaveDisable = true;
            setPMLeaveDisable = true;
        } else if(AMLeaveDisablePatternIsSelected){// 午前のスケジュールがしか設定されていない ⇒ PM半休しか選べない
            if(AMLeaveIsSelected){
                leave_type_selection.find('option:first').prop('selected', true);
            }
            setAMLeaveDisable = true;
            setPMLeaveDisable = false;
        } else if (PMLeaveDisablePatternIsSelected){// 午後のスケジュールがしか設定されていない ⇒ AM半休しか選べない
            if(PMLeaveIsSelected){
                leave_type_selection.find('option:first').prop('selected', true);
            }
            setAMLeaveDisable = false;
            setPMLeaveDisable = true;
        } else {// 両方選択可能の場合
            setAMLeaveDisable = false;
            setPMLeaveDisable = false;
        }
        leave_type_selection.find('option').filter(function(){ return this.value == 2}).prop('disabled',setAMLeaveDisable);
        leave_type_selection.find('option').filter(function(){ return this.value == 3}).prop('disabled',setPMLeaveDisable);
    };
    
    /**** 半休スケジュール判定用の部品群 ここまで ****/
    
    /**** 特定企業のカスタマイズ関連  ****/
    KOT_GLOBAL.KOT_LIB.SpecificCompanyCustom = {};
    
    
    //東急コミュニティー休暇区分対応：半休種別選択契機で休暇区分の選択解除
    KOT_GLOBAL.KOT_LIB.SpecificCompanyCustom.clearHolidayScheduleSelection = function(holidayScheduleSelectBoxId, halfScheduleSelectBoxId) {
      $("#" + halfScheduleSelectBoxId).change(function() {
        if ($(this).val()) {
          document.getElementById(holidayScheduleSelectBoxId).options[0].selected = true;
        }
      });
    };
    
    /**** 特定企業のカスタマイズ関連 ここまで ****/
    
    
    /**** UIUX201608用 ここから ****/
    
    KOT_GLOBAL.KOT_LIB.UIUX201608 = {};
    
    /** チェックボックスによるテーブル行操作のコントロール **/
    KOT_GLOBAL.KOT_LIB.UIUX201608.GroupCheckBoxCtrl = {};
    
    /**
     * 行操作のコントロール
     *
     * @param targetCheckboxClass  操作するチェックボックスのクラス
     * @param row  対象となるチェックボックスの存在するtrElement
     * @param changeCheckBoxStatus  trueの場合にはチェックボックスにチェックが無ければチェックする
     * @param allCheckId  全てチェック/非チェックのチェックボックスが存在する場合にはそのIdを指定する
     */
    KOT_GLOBAL.KOT_LIB.UIUX201608.GroupCheckBoxCtrl.rowControl = function(targetCheckboxClass, row, changeCheckBoxStatus, allCheckId) {
        
        var status = false;
        
        //チェック状態の確認
        $(row).find("input").each(function(){
            
            if($(this).attr("type") === "checkbox"
               && $(this).hasClass(targetCheckboxClass)
               && !$(this).prop("disabled")) {
                   
               if(changeCheckBoxStatus) {
                   $(this).prop("checked", !$(this).prop("checked"));
               }
               
               status = $(this).prop("checked");
               return false;
            }
        });
        
        //trの背景色変更
        if(status) {
            $(row).addClass("htBlock-selectTable_selected");
        }
        else {
            $(row).removeClass("htBlock-selectTable_selected");
        }
        
        //全てチェック/非チェックが存在する場合は1つでもチェックが外れていればチェックを外す
        if(allCheckId) {
            var allCheck = $("#" + allCheckId);
            if(allCheck.attr("id")) {
                
                var checks = $("." + targetCheckboxClass);
                if(checks.length == checks.filter(":checked").length) {
                    
                    allCheck.prop("checked", true);
                }
                else {
                    allCheck.prop("checked", false);
                }
                
            }
        }
    };
    
    /**
     * 全チェック/非チェックの動作
     * よく同じような処理を行うので関数化した
     * 
     * @param allCheckBoxId  全てチェック/非チェックのID
     * @param targetCheckBoxClass  チェック操作対象となるチェックボックスのクラス
     */
    KOT_GLOBAL.KOT_LIB.UIUX201608.GroupCheckBoxCtrl.allCheck = function(allCheckBoxId, targetCheckBoxClass) {
        
        var allCheckBox = $("#" + allCheckBoxId);
        var checked = allCheckBox.prop("checked");
        $("." + targetCheckBoxClass).each(function(){
            
            if($(this).prop("disabled")) {
                return true;
            }
            
            if(checked) {
                $(this).prop("checked", true);
                $(this).closest("tr").addClass("htBlock-selectTable_selected");
            }
            else {
                $(this).prop("checked", false);
                $(this).closest("tr").removeClass("htBlock-selectTable_selected");
            }
        });
        
    };
    
    /**** UIUX201608用 ここまで ****/
    
    
    
})();



/*
 * 各ページで個別に行っていた値変更時の強調表示をプラグイン化
 * セレクタにはdocument.bodyを指定してください。
 * 
 * $(セレクタ).emphasizeOnChangeValue({targets : [],customTargets : []})
 * 
 * 仕様　http://test.h-t.co.jp/trac/kot2/wiki/CodingStandardsSpecificationsForEditEmphasize
 * 
 ** --emphasizeOnChangeValue の定義 ここから------------------------------------------------------------------------------------------------------------------------------------ */
(function($) {
    
    /* ----内部で使用する強調表示用関数---ここから-------------------------------------------------------------------------------------------------------------------------------- */
    
    //基本設定の変更箇所が元に戻った場合に背景色を戻す
    //対象objの直近のTRまたはDIVまでさかのぼる
    //それ以下のTD,INPUTの背景色を変更する。
    //直近の親がDIVだった場合にはDIV内の色しか変わらないので注意
    //元の値から変わった場合にはtrue,元の値に戻ったらfalse
    //複数のInput,Selectがあり、どれかが変更されている場合にはtrue
    function _emphasizeOnChangeValue(obj,valueMap,keyType) {
        //そのobjから親のTRまたはDIVまで遡る
        var val;
        if(obj.tagName == "SELECT") {
            val = obj.options[obj.selectedIndex].value;
        }
        else if(obj.tagName == "INPUT" && obj.type == "checkbox") {
            val = obj.checked;
        }
        else {
            val = obj.value;
        }
        
        var parent = obj.parentNode;
        //オブジェクトの大本の親（TRかDIV）までさかのぼる
        while (parent.tagName != null && parent.tagName != "TR" && parent.tagName != "DIV") {
            parent = parent.parentNode;
        }
        var child = parent.childNodes;
        var checkResult = false;
        
        //操作したオブジェクトの値が、画面ロード時の値で有れば以下の処理を進める
        if((keyType == "id" && val == valueMap[obj.id]) || (keyType == "name" && val == valueMap[obj.name])) {
            //まずは配下の値をチェックする必要がある。
            //同じ親内の全ての値をチェックしてOKであれば背景色を戻すので
            //2回回さなくてはいけない
            var okFlag = true;
            
            //TRは以下のTDを調査する
            if(child != null && child.length > 0) {
                for(i = 0; i < child.length; i++) {
                    if(parent.tagName == "TR") {
                        var checkGroundChild = child[i].childNodes;
                        if(checkGroundChild != null && checkGroundChild.length > 0) {
                            for(j = 0; j < checkGroundChild.length; j++) {
                                //さらに下にDIVやSpanが有った場合にはその中も調査
                                if(!_dicideChangeObjChild(checkGroundChild[j],valueMap,keyType,false,null)) {
                                    okFlag = false;
                                    break;
                                }
                            }
                        }
                    }
                    //DIVの場合は配下を確認
                    //select,textのチェックの際にnullやundefinedを外しているのはロード時に連想配列に突っ込まない為(nullやundefinedであれば変更されていない)
                    if(parent.tagName == "DIV") {
                        if(child != null && child.length > 0) {
                            //さらに下にDIVやSpanが有った場合にはその中も調査
                            if(!_dicideChangeObjChild(child[i],valueMap,keyType,false,null)) {
                                okFlag = false;
                                break;
                            }
                        }
                    }
                }
                if(okFlag) {
                    if(parent.tagName == "TR") {
                        for(i = 0; i < child.length; i++) {
                            if(child[i] != null && child[i].tagName != null && child[i].tagName == "TD") {
                                child[i].style.backgroundColor = "#F5F5F5";
                                var grandChild  = child[i].childNodes;
                                if(grandChild != null && grandChild.length > 0) {
                                    for(j = 0; j < grandChild.length; j++) {
                                        _dicideChangeObjChild(grandChild[j],valueMap,keyType,true,"return");
                                    }
                                }
                                checkResult = false;
                            }
                        }
                    }
                    //DIVは以下のINPUTを調査する
                    else if(parent.tagName == "DIV") {
                        parent.style.backgroundColor = "#F5F5F5";
                        for(i = 0; i < child.length; i++) {
                            _dicideChangeObjChild(child[i],valueMap,keyType,true,"return");
                        }
                        checkResult = false;
                    }
                }
                else {
                    checkResult = true;
                }
            }
        }
        else {
            //TRは以下のTDを調査する
            if(child != null && child.length > 0) {
                if(parent.tagName == "TR") {
                    for(i = 0; i < child.length; i++) {
                        if(child[i] != null && child[i].tagName != null && child[i].tagName == "TD") {
                            child[i].style.backgroundColor = "#FFFF99";
                            var grandChild  = child[i].childNodes;
                            if(grandChild != null && grandChild.length > 0) {
                                for(j = 0; j < grandChild.length; j++) {
                                    _dicideChangeObjChild(grandChild[j],valueMap,keyType,true,"change");
                                }
                            }
                            checkResult = true;
                        }
                    }
                }
                else if(parent.tagName == "DIV") {
                    parent.style.backgroundColor = "#FFFF99";
                    for(i = 0; i < child.length; i++) {
                        _dicideChangeObjChild(child[i],valueMap,keyType,true,"change");
                    }
                    checkResult = true;
                }
            }
        }
        return checkResult;
    }
    
    //一つのTD用変更確認
    //元の値から変わった場合にはtrue,元の値に戻ったらfalse
    //同じTD内に複数のInput,Selectがあり、どれかが変更されている場合にはtrue
    function _emphasizeOnChangeValueForOneCell(obj,valueMap,keyType) {
        var val;
        if(obj.tagName == "SELECT") {
            val = obj.options[obj.selectedIndex].value;
        }
        else if(obj.tagName == "INPUT" && obj.type == "checkbox") {
            val = obj.checked;
        }
        else {
            val = obj.value;
        }
        
        //そのobjから親のTDまで遡る
        var parent = obj.parentNode;
        while (parent.tagName != null && parent.tagName != "TD") {
            parent = parent.parentNode;
        }
        var child = parent.childNodes;
        var checkResult = false;
        
        //操作したオブジェクトの値が、画面ロード時の値で有れば以下の処理を進める
        if((keyType == "id" && val == valueMap[obj.id]) || (keyType == "name" && val == valueMap[obj.name])) {
            //まずは配下の値をチェックする必要がある。
            //同じ親内の全ての値をチェックしてOKであれば背景色を戻すので
            //2回回さなくてはいけない
            var okFlag = true;
             
            //TDの配下を調査する
            //select,textのチェックの際にnullやundefinedを外しているのはロード時に連想配列に突っ込まない為(nullやundefinedであれば変更されていない)
            if(child != null && child.length > 0) {
                for(i = 0; i < child.length; i++) {
                    if(!_dicideChangeObjChild(child[i],valueMap,keyType,false,null)) {
                        okFlag = false;
                        break;
                    }
                }
                if(okFlag) {
                    parent.style.backgroundColor = "#F5F5F5";
                    for(i = 0; i < child.length; i++) {
                        _dicideChangeObjChild(child[i],valueMap,keyType,true,"return");
                    }
                    checkResult = false;
                }
                else {
                    //NGであれば色を変更する必要がある
                    parent.style.backgroundColor = "#FFFF99";
                    checkResult = true;
                }
            }
        }
        else {
            //TRは以下のTDを調査する
            if(child != null && child.length > 0) {
                parent.style.backgroundColor = "#FFFF99";
                for(i = 0; i < child.length; i++) {
                    _dicideChangeObjChild(child[i],valueMap,keyType,true,"change");
                }
                checkResult = true;
            }
        }
        return checkResult
    }
    
    /*td内の子要素を判定する
    * checkのみの場合値が変更された状態であればFalseを返す
    * obj　判定する要素の子要素　
    * valueMap　画面内の入力値の初期値の連想配列
    * keyType　連想配列のキーがidかNameか
    * changeColor　背景色を変更する場合 true チェックのみはfalse
    * changeType　背景色を変更する場合に返るのか戻すのか changeとreturnの文字列
    */
    function _dicideChangeObjChild(obj,valueMap,keyType,changeColor,changeType) {
        //値の判定のみ行う場合
        if(!changeColor) {
            var selectVal;
            var returnResult = true;
            //select,textのチェックの際にnullやundefinedを外しているのはロード時に連想配列に突っ込まない為(nullやundefinedであれば変更されていない)
            if(obj != null && obj.tagName != null && (obj.tagName == "INPUT" || obj.tagName == "SELECT")) {
                if(obj.tagName == "SELECT") {
                    selectVal = obj.options[obj.selectedIndex].value;
                    if((keyType == "id" && valueMap[obj.id] != null && valueMap[obj.id] != undefined && selectVal != valueMap[obj.id]) ||
                       (keyType == "name" && valueMap[obj.name] != null && valueMap[obj.name] != undefined && selectVal != valueMap[obj.name])) {
                        returnResult = false;
                    }
                }
                if((obj.tagName == "INPUT" && obj.type == "text") ||
                   (obj.tagName == "INPUT" && obj.type == "password") ||
                   (obj.tagName == "INPUT" && obj.type == "radio" && obj.checked)) {
                    if((keyType == "id" && valueMap[obj.id] != null && valueMap[obj.id] != undefined && obj.value != valueMap[obj.id]) ||
                       (keyType == "name" && valueMap[obj.name] != null && valueMap[obj.name] != undefined &&  obj.value != valueMap[obj.name])) {
                        returnResult = false;
                    }
                }
                if(obj.tagName == "INPUT" && obj.type == "checkbox") {
                    if((keyType == "id" && obj.checked != valueMap[obj.id]) ||
                       (keyType == "name" && obj.checked != valueMap[obj.name])) {
                        returnResult = false;
                    }
                }
            }
            else if(obj != null && obj.tagName != null && (obj.tagName == "DIV" || obj.tagName == "SPAN")){
                //DIVやSPANの場合にはさらに子要素をチェックする
                var objChild = obj.childNodes;
                if(objChild != null && objChild.length > 0) {
                   for(var childCount = 0; childCount < objChild.length; childCount++) {
                       //配下の値が変更されていた場合のみFalseを返す。
                       if(!_dicideChangeObjChild(objChild[childCount],valueMap,keyType,changeColor,changeType)) {
                           returnResult = false;
                       }
                   }
                }
            }
            return returnResult;
        }
        else {
            //配下のINPUTの色を変更する場合
            if(obj != null && obj.tagName != null && obj.tagName == "INPUT") {
                if(obj.type == "radio" || obj.type == "checkbox") {
                    if(changeType == "change") {
                        obj.style.backgroundColor = "#FFFF99";
                    }
                    else if(changeType == "return") {
                        obj.style.backgroundColor = "#F5F5F5";
                    }
                }
            }
            else if(obj != null && obj.tagName != null && (obj.tagName == "DIV" || obj.tagName == "SPAN")) {
                //DIVやSPANの場合にはさらに子要素をチェックする
                var objChild = obj.childNodes;
                if(objChild != null && objChild.length > 0) {
                    _dicideChangeObjChild(objChild[childCount],valueMap,keyType,changeColor,changeType);
                }
            }
        }
    }
    
    /* ----内部で使用する強調表示用関数---ここまで---------------------------------------------------------------------------------------------------------------------------------------- */
    
    
    /* ----プラグインの実処理 コンストラクタ---ここから-------------------------------------------------------------------------------------------------------------------------------- */
    
    /*
     * 必ずoptionを渡してください。
     */
    function EmphasizeOnChangeValue(element, options) {
        this.baseElement = null;
        //設定
        this.settings = {
                /*
                * 【プロパティ】
                * targets 
                * 
                * 【説明】
                * 対象となる要素をオブジェクトの配列で指定
                *  {
                *      selector : 要素をセレクトする際に使用する文字列 　テキストボックスなら"input:text"など
                *      keyCode : 対象とするキー "id" or "name"
                *      type : 強調方法 "row" or "cell"（行ごと強調したい場合は"row", TD1つだけ強調したい場合は"cell"
                *  }
                */
                targets : [],
                
                /*
                 * 【プロパティ】
                 * customTargets 
                 * 
                 * 【説明】
                 * 強調処理をカスタムで行いたい要素をオブジェクトの配列で指定
                 * groupSelectorで指定した内容を元にtargetsプロパティに設定したkeyCodeで処理をします
                 * 
                 *  {
                 *      groupSelector :  カスタム処理をしたい要素が含まれるグループのセレクタ文字列を指定（この文字列を元にtargetsを検索して、keyCodeを取得したりします）
                 *      keyValue : 要素の id または name を文字列で指定  ("paid_holiday_assign_month"など）
                 *      onFocus : フォーカス時に行う処理(何もしない場合は指定しない) 【引数 obj : 対象のオブジェクト, valueMapOnLocad : ロード時の値マップ, keyCode : "name" or "id"】
                 *      onChange : 変更時に行う処理   【引数 obj : 対象のオブジェクト, valueMapOnLocad : ロード時の値マップ, keyCode : "name" or "id"】
                 *  }
                 */
                customTargets : []
        };
        //初期値マップ
        this.valueMapOnLoad = {};
        //セレクタのマップ
        this.selectorMap = {};
        //カスタム処理を行う対象
        this.customTargetMap = {};
        this.init(element, options);
    }
    
    $.extend(EmphasizeOnChangeValue.prototype,{
        //初期化
        init : function (element, options){
            this.baseElement = $(element);
            //設定を拡張
            $.extend(this.settings,options);
            this.setSelectorMap();
            this.setCustomTargetMap();
            this.presetValues();
            this.setForcusEvents();
            this.setChangeEvents();
        },
        //targetsを連想配列化
        setSelectorMap : function() {
            for(var i = 0,len = this.settings.targets.length; i < len; i++) {
                var selector = this.settings.targets[i];
                if(!selector) {
                    continue;
                }
                var targetSelectorString = selector.selector.toLowerCase();
                selector.selector = targetSelectorString;
                this.selectorMap[targetSelectorString] = selector;
            }
        },
        //customTargetsを連想配列化
        setCustomTargetMap : function() {
            for(var i = 0,len = this.settings.customTargets.length; i < len; i++) {
                var customTarget = this.settings.customTargets[i];
                var groupSelector = customTarget.groupSelector.toLowerCase();
                if(!groupSelector) {
                    continue;
                }
                var array = this.customTargetMap[groupSelector];
                if(array == null) {
                    array = [];
                }
                customTarget.groupSelector = groupSelector;
                array.push(customTarget);
                this.customTargetMap[groupSelector] = array;
            }
        },
        //事前に値を格納する必要がある要素
        presetValues : function() {
            var self = this;
            for(var key in this.selectorMap) {
                if(this.selectorMap.hasOwnProperty(key)) {
                    var target = this.selectorMap[key];
                    if(target.selector.toLowerCase() === "input:radio") {
                        $("input:radio:checked", self.baseElement).each(function(){
                            self.setValueMap(target, this);
                        });
                    }
                    else if(target.selector.toLowerCase() === "input:checkbox") {
                        $(target.selector, self.baseElement).each(function() {
                            self.setValueMap(target, this);
                        });
                    }
                }
            }
        },
        //全体のフォーカスイベントを設定
        setForcusEvents : function() {
            var self = this;
            for(var key in this.selectorMap) {
                if(this.selectorMap.hasOwnProperty(key)) {
                    var target = this.selectorMap[key];
                    if(this.isForcusNeed(target)) {
                        self.setOneForcusEvent(target, this.baseElement);
                    }
                }
            }
        },
        //セレクタごとのフォーカスイベントを設定
        setOneForcusEvent : function(target, baseElement) {
            var self = this;
            $(target.selector, baseElement).on("focus", function(){
                
                if(!self.executeCustomFocusEvent(target, this)) {
                    //カスタムイベントが無い場合は通常処理
                    var onLoadValue = self.getTargetLoadValue(target,this);
                    if(onLoadValue === null || onLoadValue === undefined) {
                        self.setValueMap(target, this);
                    }
                }
            });
        },
        //カスタムフォーカス処理があれば行い trueを返す、無ければfalse
        executeCustomFocusEvent : function(target,obj) {
            var customs = this.customTargetMap[target.selector];
            if(customs && customs.length > 0) {
                for(var i = 0, len = customs.length; i < len; i++) {
                    //処理が無い場合
                    if(!customs[i] || !customs[i].onFocus) {
                        continue;
                    }
                    
                    if((target.keyCode === "name" && obj.name === customs[i].keyValue) 
                        || (target.keyCode === "id" && obj.id === customs[i].keyValue)) {
                        customs[i].onFocus(obj, this.valueMapOnLoad, target.keyCode);
                        return true;
                    }
                }
            }
            return false;
        },
        //変更イベントを設定
        setChangeEvents : function() {
            var self = this;
            for(var key in this.selectorMap) {
                if(this.selectorMap.hasOwnProperty(key)) {
                    var target = this.selectorMap[key];
                    
                    var eventKey = "change";
                    if(target.selector === "input:checkbox") {
                        eventKey = "click";
                    }
                    self.setOneChangeEvent(target, eventKey, this.baseElement);
                }
            }
        },
        //セレクタごとのフォーカスイベントを設定
        setOneChangeEvent : function(target, eventKey, baseElement) {
            var self = this;
            $(target.selector, baseElement).on(eventKey, function(){
                if(!self.executeCustomChangeEvent(target,this)) {
                    //カスタムイベントが無い場合は通常処理
                    self.emphasize(target, this);
                }
            });
        },
        //カスタム変更処理があれば行いtrue, 無ければfalse
        executeCustomChangeEvent : function(target,obj) {
            var customs = this.customTargetMap[target.selector];
            if(customs && customs.length > 0) {
                for(var i = 0, len = customs.length; i < len; i++) {
                    //処理が無い場合
                    if(!customs[i] || !customs[i].onChange) {
                        continue;
                    }
                    
                    if((target.keyCode === "name" && obj.name === customs[i].keyValue) 
                        || (target.keyCode === "id" && obj.id === customs[i].keyValue)) {
                        customs[i].onChange(obj, this.valueMapOnLoad, target.keyCode);
                        return true;
                    }
                }
            }
            return false;
        },
        setValueMap : function(target, obj) {
            if(target.keyCode === "id") {
                this.valueMapOnLoad[obj.id] = this.getTargetObjctValue(obj);
            }
            else if(target.keyCode == "name") {
                this.valueMapOnLoad[obj.name] = this.getTargetObjctValue(obj);
            }
        },
        getTargetLoadValue : function(target, obj) {
            if(target.keyCode === "id") {
                return this.valueMapOnLoad[obj.id];
            }
            else if(target.keyCode === "name") {
                return this.valueMapOnLoad[obj.name];
            }
            return null;
        },
        getTargetObjctValue : function(obj) {
            if(obj.tagName === "INPUT" && obj.type === "checkbox") {
                return obj.checked;
            }
            else if(obj.tagName === "SELECT") {
                return obj.options[obj.selectedIndex].value;
            }
            else {
                return obj.value;
            }
        },
        isForcusNeed : function(target) {
            if(target.selector === "input:text" ||
               target.selector === "input:password" ||
               target.selector === "select" ||
               target.selector === "textarea") {
                return true;
            }
            return false;
        },
        emphasize : function(target, obj) {
            //直近TRのクラスが”single_td_check”だった場合はセル単位となる
            var isCell = false;
            var parentTR = $(obj).closest("TR");
            if(parentTR) {
                parentTR = $(parentTR).get(0);
                if(parentTR.className === "single_td_check") {
                    isCell = true;
                }
            }
            
            if(isCell) {
                _emphasizeOnChangeValueForOneCell(obj, this.valueMapOnLoad, target.keyCode);
            }
            else if(target.type === "row") {
                _emphasizeOnChangeValue(obj, this.valueMapOnLoad, target.keyCode);
            }
            else if(target.type === "cell") {
                _emphasizeOnChangeValueForOneCell(obj, this.valueMapOnLoad, target.keyCode);
            }
        },
        //外から呼ぶ用
        emphasizeCallByOutSide : function(obj, keyCode, type) {
            if(type === "row") {
                _emphasizeOnChangeValue(obj, this.valueMapOnLoad, keyCode);
            }
            else if(type === "cell") {
                _emphasizeOnChangeValueForOneCell(obj, this.valueMapOnLoad, keyCode);
            }
        }
    });
    
    /* ----プラグインの実処理 コンストラクタ---ここまで-------------------------------------------------------------------------------------------------------------------------------- */
    
    /* ----プラグイン定義---ここから--------------------------------------------------------------------------------------------------------------------------------------------- */
    
    $.fn.emphasizeOnChangeValue = function(options) {
        if(this.get(0) === document.body) {
            //TODO : 外からメソッドだけ呼べるようにしたい
            return new EmphasizeOnChangeValue(this,options);
        }
        else {
            return this.each(function(){
                new EmphasizeOnChangeValue(this,options);
            });
        }
    };
    
    /* ----プラグイン定義---ここまで-------------------------------------------------------------------------------------------------------------------------------------------- */
    
})(jQuery);

/* --emphasizeOnChangeValue の定義 ここまで------------------------------------------------------------------------------------------------------------------------------------ */