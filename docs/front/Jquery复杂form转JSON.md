# Jquery复杂form转JSON

需求：需要转成的JSON数据格式如下：

![image-20220608134257060](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/front/img/image-20220608134257060.png)

前端表单包括input和checkbox：

![image-20220608134450827](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/front/img/image-20220608134450827.png)

解决方案：

1. 名称、背景、总结说明部分都是普通文本输入，不赘述。

2. 原理的处理：

   html:

   ````html
   <div class="form-group">
       <label id="theory">原理</label>
       <div class="custom-control custom-checkbox" th:each="theory,state:${theoryList}">
           <input type="checkbox" class="custom-control-input" th:id="${theory.theoryId}" name="check" th:value="${theory.theoryName}">
           <label class="custom-control-label" th:for="${theory.theoryId}" th:text="${theory.theoryName}"></label>
       </div>
   </div>
   ````

   用themeleaf绑定一个value属性，用于动态获取checkbox的内容。【坑】如果不绑定value属性的话，直接调用.val()方法得到的值是'on'，表示该checkbox被选中。

   js:

   ````javascript
   let theories = {};
   $("input[name='check']:checked").each(function (index) {
       theories[index+1] = {"theoryId":$(this).attr("id"),"theoryName":$(this).val()};
   })
   plan["trainingTheories"] = theories;
   ````

3. 步骤的处理：

   html:

   ````html
   <div id="part">
       <div id="partControll">
           <label margin-right="25px">步骤</label>&nbsp;&nbsp;
           <button type="button" class="btn btn-sm btn-primary" margin-left="10px" onclick="addPart()">+</button>&nbsp;
           <button type="button" class="btn btn-sm btn-primary" margin-left="10px" onclick="deletePart()">-</button>
       </div>
       <div id="partDetails">
           <div id="part1" name="partStep">
               <div class="form-row">
                   <div class="form-group col-md-6">
                       <input type="text" name="partOrder" class="form-control" placeholder="#1" order="1" readonly>
                   </div>
                   <div class="form-group col-md-6">
                       <input type="text" class="form-control" placeholder="步骤名称" name="partName">
                   </div>
               </div>
               <div class="form-group">
                   <textarea class="form-control" rows="3" placeholder="步骤详情" name="partContent"></textarea>
               </div>
           </div>
       </div>
   </div>
   ````
   步骤的添加和移除的方法:

   ````javascript
   function addPart() {
           partCount++;
           $('#partDetails').append("                    <div id=\"part"+partCount+"\" name=\"partStep\">\n" +
               "                        <div class=\"form-row\">\n" +
               "                            <div class=\"form-group col-md-6\">\n" +
               "                                <input type=\"text\" name=\"partOrder\" class=\"form-control\" placeholder=\"#"+partCount+"\" order=\""+partCount+"\" readonly>\n" +
               "                            </div>\n" +
               "                            <div class=\"form-group col-md-6\">\n" +
               "                                <input type=\"text\" class=\"form-control\" placeholder=\"步骤名称\" name=\"partName\">\n" +
               "                            </div>\n" +
               "                        </div>\n" +
               "                        <div class=\"form-group\">\n" +
               "                            <textarea class=\"form-control\" rows=\"3\" placeholder=\"步骤详情\" name=\"partContent\"></textarea>\n" +
               "                        </div>\n" +
               "                    </div>")
       }
       function deletePart() {
           $('#part'+partCount).remove();
           partCount--;
       }
   ````

   js:

   ````javascript
   let parts = {};
   $("[name='partStep']").each(function () {
       parts[$(this).find("[name='partOrder']").attr("order")] = {"partName":$(this).find("[name='partName']").val(),"partContent":$(this).find("[name='partContent']").val()};
   })
   plan["trainingParts"] = parts;
   ````

​		由于partStep是一组表单，每次遍历都需要查找partStep下的元素，并将相应的值存入json数据中。



4. 完整的js如下：

````javascript
$('#submit').click(function () {
    let plan = {};
    plan["planName"] = $('#planName').val();
    plan["background"] = $('#background').val();
    plan["summary"] = $('#summary').val();
    let theories = {};
    $("input[name='check']:checked").each(function (index) {
        theories[index+1] = {"theoryId":$(this).attr("id"),"theoryName":$(this).val()};
    })
    plan["trainingTheories"] = theories;
    let parts = {};
    $("[name='partStep']").each(function () {
        parts[$(this).find("[name='partOrder']").attr("order")] = {"partName":$(this).find("[name='partName']").val(),"partContent":$(this).find("[name='partContent']").val()};
    })
    plan["trainingParts"] = parts;
    let json = JSON.stringify(plan);
    console.log(json);
})
````

