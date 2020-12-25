## 窗口函数 window function

窗口函数也就是OLAP函数（Online Analytical Processing），意思是对数据库进行实时分析处理。往常的SELECT语句
是对整张表进行查询，而窗口函数可以让我们针对整张表的某部分数据进行有选择地汇总、计算、排序。

    <窗口函数> OVER ([PARTITION BY <列名1>] --根据列名1分组，可省略
                        ORDER BY<列名2>)    --根据列名2排序
                     
举例

    SELECT product_name, product_type, sale_price, RANK() OVER (PARTITION BY product_type
                             ORDER BY sale_price) AS ranking
    FROM product 
    
窗口函数的名字叫做RANK()，该窗口函数以product_type分组，再根据sale_price对每组所有记录进行排序。
直观地，可以感觉出PARTITION BY和GROUP BY有点像，但是并不会像GROUP BY那样将每组记录汇总以减少行数，
而是不影响返回的表中数据的行数，只影响窗口函数结果是怎么被计算出来的。 PARTITION BY像是指定了一个范围，
本例中product_type就是一个window。

ORDER BY指定以哪种顺序依据哪一列进行行的排列。

## 窗口函数的种类

1. 将SUM/ MAX/ MIN等聚合函数用在窗口函数中

2. RANK/ DENSE_RANK等排序用的专用窗口函数

### 专用窗口函数

#### RANK函数

排序存在并列时会跳过之后的位次，如1，1，1，4...

#### DENSE_RANK函数

存在相同排序时不会跳过之后的位次，如1，1，1，2...

#### ROW_NUMBER

在排序时赋予唯一的连续位次，比如以上情形用该函数时，1,2,3,4...

当不指明partition by时，默认在全表内根据某列进行排序。指明时，默认在组内根据某列进行排序。

### 聚合函数在窗口函数上的应用

是一个动态的计算过程。根据id进行排序，然后逐行计算出sum_price和avg_price

    SELECT  product_id
       ,product_name
       ,sale_price
       ,SUM(sale_price) OVER (ORDER BY product_id) AS current_sum
       ,AVG(sale_price) OVER (ORDER BY product_id) AS current_avg  
    FROM product; 
    
    
## 窗口函数的应用 - 计算移动平均

以上那个动态计算的例子已经是在计算移动平均了，但是窗口函数还可以指定更加详细的汇总范围。该汇总范围成为框架frame。

    <窗口函数> OVER (ORDER BY <排序用列名>
                      ROWS n PRECEDING)
                      
    <窗口函数> OVER (ORDER BY <排序用列名>
                      ROWS BETWEEN n PRECEDING AND n FOLLOWING)
                      
PRECEDING将frame指定为自身行+“截止到之前n行”。FOLLOWING将frame指定为自身行+“截止为之后n行”。

BETWEEN 1 PRECEDING AND 1 FOLLOWING 将框架指定为前一行+自身行+后一行

    AVG(sale_price) OVER (ORDER BY product_id
                          ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS current_avg
                          
计算三行的移动平均。

**问题：第一/倒数第一怎么办？** （本身+前/后一行）/2 。同理，计算本身+前一行的sum，第一行的moving_sum就是本身。

**窗口函数的使用范围和注意事项**

- 原则上，window function只能在select语句使用

- window function OVER中的ORDER BY子句并不会影响最终结果的排序，它只是用来决定window
 function 按照什么方式计算。
 
 
 ## GROUPING 运算符
 
 ### ROLLUP - 计算合计和小计
 
 GROUP BY只能得到每个分类的小计，有时候需要计算分类的合计，用ROLLUP关键字。
 
    SELECT  product_type
       ,regist_date
       ,SUM(sale_price) AS sum_price
    FROM product
    GROUP BY product_type, regist_date WITH ROLLUP 
 
 就ROLLUP真挺好用的，可以计算每一小类的合计和总计。
 
 [练习](http://datawhale.club/t/topic/472)
 
 5.1 
 
 根据product_id排序，得到移动最大售价商品。
 
 5.2
 
    SELECT  product_id, product_name, sale_price, regist_date
    , SUM(sale_price) OVER (ORDER BY regist_date ASC) AS moving_highest
    FROM  product
    
5.3

1) 窗口函数不指定PARTITION BY的效果是什么？

window function不指定PARTITION BY就是在全表内根据ORDER BY的列计算window function

2) 为什么说窗口函数只能在SELECT子句中使用？ 

因为SQL查询是按照FROM -> WHERE -> SELECT的顺序进行的，而窗口函数的计算要based on SELECT语句的结果，所以要在同一步进行。
