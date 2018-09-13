# HomeWork
#MariaDB的查询
select sh.cityName 省,s.cityName 市,x.cityName 县  
from s_provinces x  
inner join s_provinces s on x.parentId = s.id   
inner join s_provinces sh on s.parentId = sh.id   
where sh.cityName="广东省";  

当一个表存在递归关系时，我们首先从最基层开始进行查询    
首先是县和市的查询：  
select s.cityName '市', x.cityName as '县' from s_provinces x  
  inner join s_provinces s on x.parentId = s.id;  
这里引用了 s_provinces 表里的 parendId 和 id 列  
然后就是再添加对省的关联  
select sh.cityName as '省',s.cityName as '市',x.cityName as '县' from s_provinces x  
    inner join s_provinces s on x.parentId = s.id  
    inner join s_provinces sh on s.parentId = sh.id;  
parentId 列 id 分别为省、市、县提供了互相查询的方法，之后再添加条件就可以查到想要的数据了。



#分割方法的总结
##题目要求：
###对 lagou_position 表数据进行清理，然后按照三大范式分离出 lagou_position、lagou_city、lagou_company三个表
##思路整理
1、取出公司的相关字段，company_id、company_short_name、company_full_name、company_size、financestage (注：要去除重复数据)，把这些数据转移到 lagou_company 表中。
    1) create table lagou_company as  
         select distinct t.company_id as cid,  
         t.company_short_name as short_name,  
         t.company_full_name  as full_name,  
         t.company_size       as size,  
         t.financestage  
         from lagou_position t;  
    2) 首先是先要先查询出在 lagou_position 表中和公司有关的五列信息，然后再创建 lagou_company 表将查询的信息转移到新表中。  

2、根据 p_province 表获得区id、省、市、县 (区) 信息。  
    1) select d.id, p.cityName as province, c.cityName as city, d.cityName as district from  
         (select * from p_province where depth=3) d  
         join p_province c on d.parentId = c.id and c.depth=2  
         join p_province p on c.parentId = p.id and p.depth=1;  
    2) 在 p_province 表中用链接查询出区县地区的数据。  

3、再根据 p_province 表获取市id、省、市、null 的信息。  
    1) select c.id, p.cityName as province, c.cityName as city, null as district from  
         (select * from p_province where depth=2) c  
         join p_province p on c.parentId = p.id and p.depth = 1;  
    2) 根据 p_province 表查询出市地区的数据  

4、再根据 2、3 的步骤的查询出所有的地址信息，然后插入到 lagou_city 表中。  
    1) create table lagou_city as  
         select d.id, p.cityName as province, c.cityName as city, d.cityName as district from  
         (select * from p_province where depth=3) d  
         join p_province c on d.parentId = c.id and c.depth=2  
         join p_province p on c.parentId = p.id and p.depth=1  
         union  
         select c.id, p.cityName as province, c.cityName as city, null as district from  
         (select * from p_province where depth=2) c  
         join p_province p on c.parentId = p.id and p.depth = 1;  
    2) 将查询出来的 id( 市id 和 县id )、省、市、县 数据转移到 lagou_city 表中  

5、然后就是对 lagou_position 的清理，首先删除步骤 1 分离出去的公示表相对的列。  
    1) alter table lagou_position_bk drop company_size;  
       alter table lagou_position_bk drop company_full_name;  
       alter table lagou_position_bk drop company_short_name;  
       alter table lagou_position_bk drop company_id;  
       alter table lagou_position_bk drop financestage;  
    2) 为了实现数据库三大范式的设计，对数据更好的维护和数据冗余等情况。在转移好数据后我们就要把重复的列清除掉。

6、其次再删除 lagou_position 表 district 列，因为 district 作为保存 县 (区) 信息的一列，不应该在 lagou_position 表中出项，该表是作为保存员工工作地的和工作信息的，也是考虑到会出现数据冗余的情况，因为我们在 lagou_city 表已经保存了完整的地区信息。