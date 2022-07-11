### 1. 需求

项目后台使用springboot，用thymeleaf渲染页面。原本页面展示一个列表的数据，需要实现按时间对列表数据进行排序：

![image-20220709165326589](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/CI/img/image-20220709165326589.png)

### 2. 解决方案

利用thymeleaf的fragment来替换需要刷新的内容，具体实现如下：

#### 2.1 后端

先创建一个新的fragment页面，保存刷新后后端返回的数据。

原页面如下，使用th:each遍历得到.card.planItem的列表

![image-20220709165940890](D:\ResilioSync\myBlog\docs\front\img\image-20220709165940890.png)

新创建的fragment页面

![image-20220709170029608](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/CI/img/image-20220709170029608.png)

在controller中增加对应的接口：

````java
  @GetMapping("index/timeAsc")
  public String findAllByTimeAsc(Model model){
    List<TrainingPlan> allTrainingPlans = trainingPlanService.findAllOrderByCreatedTimeAsc();
    model.addAttribute("planList",allTrainingPlans);
    return "fragments/plans::sortedPlan";
  }

  @GetMapping("index/timeDesc")
  public String findAllByTimeDesc(Model model){
    List<TrainingPlan> allTrainingPlans = trainingPlanService.findAllOrderByCreatedTimeDesc();
    model.addAttribute("planList",allTrainingPlans);
    return "fragments/plans::sortedPlan";
  }
````

返回 return "fragments/plans::sortedPlan";

#### 2.2 前端

````javascript
function sortDesc() {
    let url = "http://".concat(serverAdd,"/trainingPlan/index/timeDesc")
    $.ajax({
        type: "get",
        url: url,
        dataType: "text",
        success: function (data) {
            $("#planItems").empty();
            $("#planItems").html(data);
        }
    });
}
function sortAsc() {
    let url = "http://".concat(serverAdd,"/trainingPlan/index/timeAsc")
    $.ajax({
        type: "get",
        url: url,
        dataType: "text",
        success: function (data) {
            $("#planItems").empty();
            $("#planItems").html(data);
        }
    });
}
````

将.card.planItem的父容器#planItems清空，并将后端返回的fragment页面填入html内容。

