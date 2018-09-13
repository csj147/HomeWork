# HomeWork
#MariaDB的查询


select sh.cityName 省,s.cityName 市,x.cityName 县 
from s_provinces x 
inner join s_provinces s on x.parentId = s.id 
inner join s_provinces sh on s.parentId = sh.id 
where sh.cityName="广东省"; 

当一个表存在递归关系时，我们首先从最基层开始进行查询 
*我们先关联 县和市 的查询 select * from s_provinces x inner join s_provinces s on x.parentId=s.id;
*这样引用 s_provinces 表里的 parendId 和 id 关联 定位到 子和父的关系。
*然后就可以按照自已的需要进行查询了。