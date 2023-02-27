# SpringBoot项目-商品搜索

## 9 商品搜索

### 9.1 关于商品的模糊搜索

---

#### 9.1.1 后端-持久层

#### 1.编写sql

```sql
SELECT id,title,sell_point,price,image
FROM t_product 
WHERE STATUS = 1
AND title LIKE '%${title}%' 
ORDER BY priority DESC;
```

---

#### 2.编写Mapper接口的抽象方法

```java
    /**
     * 按照输入的标题查找
     * @param title
     * @return
     */
    List<Product> findProductByTitle(String title);
```

---

#### 3.编写Mapper接口的映射文件

```xml
<!--    List<Product> findProductByTitle(String title);-->
    <select id="findProductByTitle" resultType="com.zxl.store.entity.Product">
        SELECT id,title,sell_point,price,image
        FROM t_product
        WHERE STATUS = 1
        AND title LIKE '%${title}%'
        ORDER BY priority DESC
    </select>
```

---

### 9.1.2 后端-业务层

#### 1.处理异常

---

#### 2.编写业务层抽象方法

```java
    /**
     * 按照标题查询
     * @param title
     * @param pageNum
     * @param pageSize
     * @return
     */
    PageInfo<Product> findProductByTitle(String title,Integer pageNum, Integer pageSize);
```

---

#### 3.编写业务层逻辑

```java
    @Override
    public PageInfo<Product> findProductByTitle(String title, Integer pageNum, Integer pageSize) {
        //开启分页功能
        PageHelper.startPage(pageNum,pageSize);
        //查询结果
        List<Product> res = productMapper.findProductByTitle(title);
        //返回结果
        return new PageInfo<>(res);
    }
```

---

### 9.1.3 后端-控制层

---

#### 1.处理异常

---

#### 2.设计请求

请求路径：/product/findWithTitle 

请求参数：Integer pageNum，Integer pageSize，String title

请求类型：get

响应类型：JsonResult<PageInfo< Product>>

---

#### 3.处理请求，编写控制层方法

```java
    @RequestMapping(value = "/{pageNum}/{pageSize}/{title}",method = RequestMethod.GET)
    public JsonResult<PageInfo<Product>> findWithTitle(@PathVariable("pageNum") Integer pageNum,
                                                       @PathVariable("pageSize") Integer pageSize,
                                                       @PathVariable("title") String title){
        PageInfo<Product> res = productService.findProductByTitle(title, pageNum, pageSize);
        return new JsonResult<>(OK,res);
    }
```

----

#### 4.前端页面

具体请参考github

```javascript
		<script type="text/javascript">
			//获取标题
			var title = getOne();
			title = title.substring(0,title.indexOf("&"));

			function getUrlParam(name) {
				var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
				var r = window.location.search.substr(1).match(reg);
				if (r != null)
					return unescape(r[2]);
				return null;
			}

			var num = getUrlParam("pageNum");

			//为了查询设置的全局参数
			var pageNum = 0;
			var pageSize = 0;
			var prePage = 0;
			var nextPage = 0;
			var navigatepageNums = [];

			//记录fid
			var fid = 0;

			//根据分页条的选择的页数进行查询
			function PaginationListSelect(num,size,res) {
				//进行查询的关键字
				let title =  $("#search").val()
				llocation.href = "/web/search.html?title=" + title + "&pageNum=" + num + "&pageSize=" + size;
				//填充分页条信息
				addPaginationData(res);
			}

			//按照标题查询数据
			function searchByTitle(pageNum,pageSize,title){
				$.ajax({
					url:"/product/" + pageNum + "/" + pageSize + "/" + title,
					dataType:"json",
					success:function (res) {
						if(res.state==200){
							if(res.data.list.length!=0){
								$("#resultTest").html("\""+ title + "\"");
								addData(res);
							}else{
								$("#resultTest").html("\""+ title + "\"");
								$("#errmsg").html("目前该商品并不在库");
							}
						}
					}
				});
			}

			//填充数据
			function addData(res){
				//清空目标内容
				$("#productList").empty();
				$("#PaginationList").empty();
				let productListStr = "";
				//获取分页信息中商品的长度
				let dataLength = res.data.list.length;
				//填充商品到信息页面
				for (let i = 0; i < dataLength; i++) {
					let product = res.data.list[i]
					productListStr = "<div class=\"col-md-3\">"
							+ "<div class=\"goods-panel\">"
							+ "<img src=.." + product.image + "collect.png" + "  class=\"img-responsive\" />"
							+ "<p>￥" + product.price  + ".00" + "</p>"
							+ "<div class=\"text-row-3\">"
							+ "<a href=product.html?pid=" + product.id
							+ "><small>"
							+ product.title
							+ "</small></a></div>"
							+ "<span style='padding-right: 10px'>"
							+ "<a href='javascript:void(0)' onclick='addToCollect(#{id})' id='product#{num}' class='btn btn-default btn-xs add-fav'><span class='fa fa-heart-o'></span>加入收藏</a>"
							+ "</span>"
							+ "<span style='padding-right: 10px'>"
							+ "<a href='javascript:void(0)' onclick='addCollectToCart(#{id},#{price})' class=\"btn btn-default btn-xs add-cart\"><span class=\"fa fa-cart-arrow-down\"></span>加入购物车</a>"
							+ "</span></div></div>"

					productListStr = productListStr.replaceAll("#{id}",product.id);
					productListStr = productListStr.replaceAll("#{price}",product.price);
					$("#productList").append(productListStr)
				}
				//填充分页条信息
				addPaginationData(res);
			}
			//填充分页条信息
			function addPaginationData(res) {
				//重新填充分页条
				//将数据返回的部分需要数据进行填充至全局参数
				pageNum = res.data.pageNum //当前页
				pageSize = res.data.pageSize  //每页显示数
				prePage =  res.data.prePage //上一页
				nextPage = res.data.nextPage //下一页
				navigatepageNums = res.data.navigatepageNums //分页栏的数字

				let firstPage = "<a id='first' href='#' onclick='PaginationListSelect(prePage,pageSize)' style='padding-right: 8px'>上一页</a>"
				let lastPage = "<a id='end' href='javascript:void(0)' onclick='PaginationListSelect(nextPage,pageSize)' style='padding-right: 8px'>下一页</a>"
				let PaginationListStr = "";
				//判断是否是第一页
				if (res.data.isFirstPage){ //为true表示当前是第一页
					firstPage = "<a id='first' href='javascript:void(0)' "
							+ "style='opacity: 0.2;padding-right: 8px;color: black'>上一页</a>"
					PaginationListStr += firstPage;
				}else { //为false表示当前不是第一页
					PaginationListStr += firstPage;
				}
				//填充分页的页码数
				for (let i = 0; i < navigatepageNums.length; i++) {
					//当前页的页码
					let nowNum = navigatepageNums[i]
					if (nowNum === pageNum){ //相等表示i的次数和当前也相同，对页数显示做变化
						PaginationListStr += "<a href='javascript:void(0)' " +
								"style='padding-right: 8px;color: black' disabled='disabled'>" + "【"  + nowNum + "】" +"</a>"
					}else {
						PaginationListStr += "<a href='javascript:void(0)' onclick='PaginationListSelect(#{nowNum},pageSize)' " +
								"style='padding-right: 8px'>" + nowNum +"</a>"
					}
					PaginationListStr = PaginationListStr.replaceAll("#{nowNum}",nowNum)

				}
				//判断是否是末页
				if (!res.data.isLastPage){ //取反为false表示当前是末页
					PaginationListStr += lastPage;
				}else { //为true表示当前是末页
					lastPage =  "<a id='end' href='javascript:void(0)' style='opacity: 0.2;padding-right: 8px;color: black'>下一页</a>"
					PaginationListStr += lastPage;
				}
				//将拼接的str串插入指定id处
				$("#PaginationList").append(PaginationListStr)
			}


			function addToCollect(pid) {
				if (confirm("确定要将此商品加入收藏吗？")) {
					$.ajax({
						url: "/favorite/addFavorite",
						type: "post",
						data: {"pid": pid},
						dataType: "json",
						success: function (res) {
							if (res.state == 200) {
								alert("收藏成功！");
								//测试？
								// alert(res.data);
							} else {
								//输出错误问题
								alert(res.message);
							}
						},
						error: function (err) {
							alert("服务器出现错误，加入购物车失败！")
						}
					})
				}
			}

			function addCollectToCart(pid,price) {
				if(confirm("确定加入购物车吗?")){
					$.ajax({
						url:"/cart/add_cart",
						type:"post",
						data:{"pid":pid,"price":price,"num":1},
						dataType:"json",
						success:function (res) {
							if(res.state==200){
								alert("加入购物车成功");
							}
						}
					});
				}
			}
			//页面加载时自动启动
			$(function () {
				searchByTitle(num,12,title);
			});
		</script>
```

---


