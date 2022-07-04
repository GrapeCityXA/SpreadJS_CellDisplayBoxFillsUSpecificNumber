# SpreadJS_CellDisplayBoxFillsUSpecificNumber
在纯前端在线表格中实现单元格显示方框填写特定个数的数值功能
# SpreadJS_CellDisplayBoxFillsUSpecificNumber

### SpreadJS 示例，单元格显示方框填写特定个数的数值（掩码输入）
该示例包括使用 SpreadJS API 的演示脚本，可用于实现单元格显示方框填写特定个数的数值（掩码输入）。
有关 SpreadJS API 的更多信息，请参阅[SpreadJS API指南]( https://demo.grapecity.com.cn/spreadjs/help/api/) 和[帮助手册]( https://help.grapecity.com.cn/pages/viewpage.action?pageId=5963808)。




### 运行步骤
1、在开始之前，请确保您已满足以下先决条件：
要运行 SpreadJS，浏览器必须支持 HTML5，客户端导入和导出 Excel 需要 IE10及以上。
请先了解 [SpreadJS 的产品使用环境]( https://www.grapecity.com.cn/developer/spreadjs/selection-guide/product-use-environment)，并申请临时部署授权激活
安装并更新NodeJS和NPM
2、克隆或下载此代码库
3、初始化控件，并运行示例脚本
#### 控件初始化
首先，创建一个新页面，并在页面上输入以下代码：
```
<!DOCTYPE html>
    <html>
    <head>
        <title>SpreadJS HTML Test Page</title>
```
2、在页面中添加对 SpreadJS 的引用。代码如下。需要注意的是，SpreadJS 提供压缩过
```
//（minified）的 JavaScript 文件和和用于调试的文件：
<script src="[Your_Scripts_Path]/gc.spread.sheets.all.xxxx.min.js" type="text/javascript"></script>
```
3、添加 CSS 文件以改变Spread.JS 的外观。默认的CSS文件名为： 
gc.spread.sheets.xxxx.css，里面包含了所有的默认样式。该 CSS 文件将会影响滚动条，筛选框及其子元素，单元格和下方标签栏的样式。引入 CSS 的代码如下：
```
//<link href="[Your_CSS_Path]/gc.spread.sheets.xxxx.css" rel="stylesheet" type="text/css"/>
```
4、添加产品授权，代码为（本地测试可以不添加）：
```
GC.Spread.Sheets.LicenseKey = "xxx";
```
5. 添加控件初始化代码。本例会在一个 id 为 “ss” 的 DOM 元素上初始化 SpreadJS：
```
<script type="text/javascript">
// Add your license
// If run this in local for testing, remove or comment below code
 GC.Spread.Sheets.LicenseKey = "xxx";

// Add your code
 window.onload = function(){
var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"),{sheetCount:3});
var sheet = spread.getActiveSheet();
 }
</script>
</head>
<body>
```
6、 创建一个 id 为 “ss” 的元素，SpreadJS 将在该 DOM 中初始化：
```
<div id="ss" style="height: 500px; width: 800px"></div>
</body>
</html>
```
#### 示例代码
```
HTML：
<p>自定义单元格-方框填写</p>
<div id='ss'></div>
CSS：
#ss{height:400px;width:100%}
p{
    color: #336699;
    text-align: center;
}
JavaScript：
var spreadNS = GC.Spread.Sheets;
        window.onload = function () {
            var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"));
            initSpread(spread);
        };
        //Custom Cell Type
        function FivePointedStarCellType() {
            this.size = 10;
        }
        FivePointedStarCellType.prototype = new spreadNS.CellTypes.Base();
        FivePointedStarCellType.prototype.paint = function (ctx, value, x, y, w, h, style, context) {
            if (!ctx) {
                return;
            }

            ctx.save();

            // draw inside the cell's boundary
            ctx.rect(x, y, w, h);
            ctx.clip();
            ctx.beginPath();

            if (value) {
                ctx.fillStyle = "orange";
            } else {
                ctx.fillStyle = "gray";
            }

            var size = this.size;
            var dx = x + w / 2;
            var dy = y + h / 2;
            ctx.beginPath();
            var dig = Math.PI / 5 * 4;
            ctx.moveTo(dx + Math.sin(0 * dig) * size, dy + Math.cos(0 * dig) * size);
            for (var i = 1; i < 5; i++) {
                ctx.lineTo(dx + Math.sin(i * dig) * size, dy + Math.cos(i * dig) * size);
            }
            ctx.closePath();
            ctx.fill();

            ctx.restore();
        };
        FivePointedStarCellType.prototype.getHitInfo = function (x, y, cellStyle, cellRect, context) {
            var xm = cellRect.x + cellRect.width / 2,
                    ym = cellRect.y + cellRect.height / 2,
                    size = 10;
            var info = { x: x, y: y, row: context.row, col: context.col, cellRect: cellRect, sheetArea: context.sheetArea };
            if (xm - size <= x && x <= xm + size && ym - size <= y && y <= ym + size) {
                info.isReservedLocation = true;
            }
            return info;
        };
        FivePointedStarCellType.prototype.processMouseUp = function (hitInfo) {
            var sheet = hitInfo.sheet;
            if (sheet && hitInfo.isReservedLocation) {
                var row = hitInfo.row, col = hitInfo.col, sheetArea = hitInfo.sheetArea;
                var newValue = !sheet.getValue(row, col, sheetArea);
                var spread = sheet.getParent();
                spread.commandManager().execute({cmd: "editCell", sheetName: sheet.name(), row: row, col: col, newValue: newValue});
                return true;
            }
            return false;
        };

        function FullNameCellType() {
        }
        FullNameCellType.prototype = new spreadNS.CellTypes.Base();
        FullNameCellType.prototype.paint = function (ctx, value, x, y, w, h, style, options) {
            if (value) {
                spreadNS.CellTypes.Base.prototype.paint.apply(this, [ctx, value.firstName + "." + value.lastName, x, y, w, h, style, options]);
            }
        };
        FullNameCellType.prototype.updateEditor = function(editorContext, cellStyle, cellRect) {
            if (editorContext) {
                editorContext.style.width=cellRect.width;
                editorContext.style.height=100;
                return {height: 100};
            }
        };
        FullNameCellType.prototype.createEditorElement = function () {
            var div = document.createElement("div");
            div.setAttribute("gcUIElement", "gcEditingInput");
            div.style.backgroundColor= "white";
            div.style.overflow= "hidden";
            var span1 = document.createElement('span');
            span1.style.display = "block";
            var span2 = document.createElement("span");
            span2.style.display = "block";
            var input1 = document.createElement("input");
            var input2 = document.createElement("input");
            var type = document.createAttribute('type');
            type.nodeValue="text";
            var clonedType = type.cloneNode(true);
            input1.setAttributeNode(type);
            input2.setAttributeNode(clonedType);
            div.appendChild(span1);
            div.appendChild(input1);
            div.appendChild(span2);
            div.appendChild(input2);
            return div;
        };
        FullNameCellType.prototype.getEditorValue = function (editorContext) {
            if (editorContext && editorContext.children.length === 4) {
                var input1 = editorContext.children[1];
                var firstName = input1.value;
                var input2 = editorContext.children[3];
                var lastName = input2.value;
                return { firstName: firstName, lastName: lastName };
            }
        };
        FullNameCellType.prototype.setEditorValue = function (editorContext, value) {
            if (editorContext && editorContext.children.length === 4) {
                var span1 = editorContext.children[0];
                span1.innerHTML="First Name:";
                var span2 = editorContext.children[2];
                span2.innerHTML="Last Name:";
                if (value) {
                    var input1 = editorContext.children[1];
                    input1.value=value.firstName;
                    var input2 = editorContext.children[3];
                    input2.value=value.lastName;
                }
            }
        };
        FullNameCellType.prototype.isReservedKey = function (e) {
            //cell type handle tab key by itself
            return (e.keyCode === GC.Spread.Commands.Key.tab && !e.ctrlKey && !e.shiftKey && !e.altKey);
        };
        FullNameCellType.prototype.isEditingValueChanged = function(oldValue, newValue) {
            if (newValue.firstName != oldValue.firstName || newValue.lastName != oldValue.lastName) {
                return true;
            }
            return false;
        };
		
		/*自定义连续方框*/
        function ContinuousBoxCellType() {
			this.html = '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:1px;border:1px solid #999" value="{3}"/>'
				+ '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:3px;border:1px solid #999" value="{4}"/>'
				+ '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:3px;border:1px solid #999" value="{5}"/>'
				+ '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:3px;border:1px solid #999" value="{6}"/>';
        }
        ContinuousBoxCellType.prototype = new spreadNS.CellTypes.Base();
		

		ContinuousBoxCellType.prototype._getPaintStartX = function(x, y, w, h, hAlign){
			var startX = x;
			var size = this._size * this._zoomCatah;
			if(this._isHorizontal){
				switch(hAlign){
					case 1:
						//center
						startX = x + w / 2 - (this._sumItemTextWidth + size * this._items.length) / 2;
						break;
					case 2:
						//right
						startX = x + w - (this._sumItemTextWidth + size * this._items.length);
						break;
					default:
						//left
						startX = x
				}
			}
			else{
				switch(hAlign){
					case 1:
						//center
						startX = x + w / 2 - (this._maxItemTextWidth + size) / 2;
						break;
					case 2:
						//right
						startX = x + w - (this._maxItemTextWidth + size + 2);
						break;
					default:
						//left
						startX = x
				}
			}
			return startX;
		}
		ContinuousBoxCellType.prototype._getPaintStartY = function(x, y, w, h, vAlign){
			var startY = y;
			var size = this._size * this._zoomCatah;
			if(this._isHorizontal){
				switch(vAlign){
					case 1:
						//center
						startY = y + h / 2 - size / 2;
						break;
					case 2:
						//bottom
						startY = y + h - size;
						break;
					default:
						//top
						startY = y
				}
			}
			else{
				switch(vAlign){
					case 1:
						//center
						startY = y + h / 2 - size * this._items.length / 2;
						break;
					case 2:
						//bottom
						startY = y + h - size * this._items.length;
						break;
					default:
						startY = y
				}

			}
			return startY;
		}
		
		
        ContinuousBoxCellType.prototype.paint = function (ctx, value, x, y, w, h, style, context) {
			var DOMURL = window.URL || window.webkitURL || window;
			var cell = context.sheet.getCell(context.row, context.col);
			var img = cell.tag();
			if (img) {
				try{
					ctx.save();
					ctx.rect(x, y, w, h);
					ctx.clip();
					ctx.drawImage(img, x + 2, y + 2)
					ctx.restore();
					cell.tag(null);
					return;
				} catch(err){
					GC.Spread.Sheets.CustomCellType.prototype.paint.apply(this, [ctx, "#HTMLError", x, y, w, h, style, context])
					cell.tag(null);
					return;
				}
			}
			var svgPattern = '<svg xmlns="http://www.w3.org/2000/svg" width="{0}" height="{1}">' +
			'<foreignObject width="100%" height="100%"><div xmlns="http://www.w3.org/1999/xhtml" style="font:{2}">'+this.html+'</div></foreignObject></svg>';

			var data = svgPattern.replace("{0}", w).replace("{1}", h).replace("{2}", style.font).replace("{3}", value.toString()[0]==null?"":value.toString()[0]).replace("{4}", value.toString()[1]==null?"":value.toString()[1]).replace("{5}", value.toString()[2]==null?"":value.toString()[2]).replace("{6}", value.toString()[3]==null?"":value.toString()[3]);
			var doc = document.implementation.createHTMLDocument("");
			doc.write(data);
			// Get well-formed markup
			data = (new XMLSerializer()).serializeToString(doc.body.children[0]);

			img = new Image();
			//var svg = new Blob([data], {type: 'image/svg+xml;charset=utf-8'});
			//var url = DOMURL.createObjectURL(svg);
			//img.src = url;
			img.src = 'data:image/svg+xml;base64,'+window.btoa(data);
			cell.tag(img);
			img.onload = function () {
				context.sheet.repaint(new GC.Spread.Sheets.Rect(x, y, w, h));
			}
			
        };
        ContinuousBoxCellType.prototype.updateEditor = function(editorContext, cellStyle, cellRect) {
            if (editorContext) {
                editorContext.style.width=cellRect.width;
                editorContext.style.height=100;
                //return {height: 200};
            }
        };
        ContinuousBoxCellType.prototype.createEditorElement = function () {
            var div = document.createElement("div");
            div.setAttribute("gcUIElement", "gcEditingInput");
            div.style.backgroundColor= "white";
            div.style.overflow= "hidden";
			div.innerHTML = '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:1px;border:1px solid #999"/>'
				+ '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:3px;border:1px solid #999"/>'
				+ '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:3px;border:1px solid #999"/>'
				+ '<input style="margin-top:2px;display:block;float:left;height:10px;font-size:10px;line-height:10px;width:10px;text-align:center;margin-left:3px;border:1px solid #999"/>';
			
			return div;
        };
        ContinuousBoxCellType.prototype.getEditorValue = function (editorContext) {
			
			/*console.log('getEditorValue');
			if(editorContext){
				return "1111";
			}*/
			
			var value = "";
			for(var i = 0; i < 4; i++){
				value+=editorContext.children[i].value;
			}
			return value;
						
			/*
            if (editorContext && editorContext.children.length === 4) {
                var input1 = editorContext.children[1];
                var firstName = input1.value;
                var input2 = editorContext.children[3];
                var lastName = input2.value;
                return { firstName: firstName, lastName: lastName };
            }
			*/
        };
        ContinuousBoxCellType.prototype.setEditorValue = function (editorContext, value) {
			/*
            if (editorContext && editorContext.children.length === 4) {
                var span1 = editorContext.children[0];
                span1.innerHTML="First Name:";
                var span2 = editorContext.children[2];
                span2.innerHTML="Last Name:";
                if (value) {
                    var input1 = editorContext.children[1];
                    input1.value=value.firstName;
                    var input2 = editorContext.children[3];
                    input2.value=value.lastName;
                }
            }
			*/
			for(var i = 0; i < 4; i++){
				if(value.toString()[i]!=null)
				editorContext.children[i].value = value.toString()[i];
			}
			
        };
        ContinuousBoxCellType.prototype.isReservedKey = function (e) {
            //cell type handle tab key by itself
            return (e.keyCode === GC.Spread.Commands.Key.tab && !e.ctrlKey && !e.shiftKey && !e.altKey);
        };
		/*自定义连续方框结束*/	
		
		
        function initSpread(spread) {
            var sheet = spread.getSheet(0);
            sheet.suspendPaint();
            sheet.setColumnWidth(0, 100);
            sheet.setColumnWidth(1, 170);

            var columnInfo = [
                { name: "result", displayName: "Result", cellType: new FivePointedStarCellType(), size: 50 },
                { name: "person", displayName: "Person", cellType: new FullNameCellType(), size: 170 },
				{ name: "test", displayName: "test", cellType: new ContinuousBoxCellType(), size: 170 }
            ];

            var source = [
                { result: true, person: {firstName:"LeBron",lastName:"James"}, test: "3333"},
                { result: false, person: { firstName: "Chris", lastName: "Bosh" }, test: 123 },
                { result: true, person: { firstName: "Dwyane", lastName: "Wade" }, test: "456" },
            ];
            sheet.setDataSource(source);
            sheet.bindColumns(columnInfo);
            sheet.resumePaint();

        };
```

#### 关于 SpreadJS
[SpreadJS]( https://www.grapecity.com.cn/developer/spreadjs) 是一款基于 HTML5 的纯前端表格控件，兼容 450 多种 Excel 公式，具备“高性能、跨平台、与 Excel 高度兼容”的产品特性。使用 SpreadJS，可直接在 Angular、 React、 Vue 等前端框架中实现高效的模板设计、在线编辑和数据绑定等功能，为最终用户提供高度类似 Excel 的使用体验。



