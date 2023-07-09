# Android基础学习_AAChartCore


---

# 1.AAChartCore
AAChartCore,是 AAChartKit 的 Java语言版本,是在流行的开源前端图表框架的基础上,封装的面向对象的,一组简单易用,极其精美的图表绘制控件  

[AAChartCore(Java版)仓库](https://github.com/AAChartModel/AAChartCore)  
[AAChartCore(Kotlin版)仓库](https://github.com/AAChartModel/AAChartCore-Kotlin)  
[AAChartCore(ios版)仓库](https://github.com/AAChartModel/AAChartKit)  
[AAChartCore(Java版)文档](https://github.com/AAChartModel/AAChartCore/blob/master/CHINESE-README.md)  

--- 

# 2.使用


## 2.1 安装


### 2.1.1 手动安装

---

1.使用git指令拉去Java版的AAChartCore，找到Demo文件  
2.将 Demo 文件下的 assets 文件拉取到项目app/src/main下  
2.将 Demo 中的名为 AAChartCoreLib 文件夹拉取到app/src/main/java/下


## 2.2 使用

### 2.2.1 layout_xml文件

---

在xml文件内定义一个或者多个AAchartView。

```xml
    <com.zxl.anan.AAChartCore.AAChartCoreLib.AAChartCreator.AAChartView
        android:id="@+id/AAChartView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />
```

### 2.2.2 Activity

chartType有如下多种

        String Column          = "column";
        String Bar             = "bar";
        String Area            = "area";
        String AreaSpline      = "areaspline";
        String Line            = "line";
        String Spline          = "spline";
        String Scatter         = "scatter";
        String Pie             = "pie";
        String Bubble          = "bubble";
        String Pyramid         = "pyramid";
        String Funnel          = "funnel";
        String Columnrange     = "columnrange";
        String Arearange       = "arearange";
        String Areasplinerange = "areasplinerange";
        String Boxplot         = "boxplot";
        String Waterfall       = "waterfall";


```java
    private void initChartView() {
        aaChartView = findViewById(R.id.AAChartView);
        //获取当前时间
        HashMap<String, Integer> calenderTime = Util.getCalenderTime();
        int year = calenderTime.get("year");
        int month = calenderTime.get("month");
        String date = Util.getYearMonthStr(year,month);
        //获取数据
        for (int i = 0; i < typeNameArr.length; i++) {
            overview_classification oc = DBManager.queryTypeSumByYearMonthKind(year, month, 0, typeNameArr[i]);
            if(oc.getTypeName() != null){
                ocList.add(oc);
            }
        }
        //获取标签名
        String strArray[] = new String[ocList.size()];
        Float dataArray[] = new Float[ocList.size()];
        for (int i = 0; i < ocList.size(); i++) {
            strArray[i] = ocList.get(i).getTypeName();
            dataArray[i] = ocList.get(i).getTotalMoney();
        }

        //
        AAChartModel aaChartModel = new AAChartModel()
                .chartType(AAChartType.Column)
                .title("月度账单总览")//设置标题
                .subtitle(date)//设置副标题
                .backgroundColor("#FFFFFF")//设置背景色白色
                .categories(strArray)   //设置标签
                .dataLabelsEnabled(true) //点击显示数据标签
                .yAxisGridLineWidth(0f)
                //数据
                .series(
                        new AASeriesElement[]{
                        new AASeriesElement()
                                .name("Out")
                                .data(dataArray),
                });
        //展示
        aaChartView.aa_drawChartWithChartModel(aaChartModel);
    }

```
