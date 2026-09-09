非洲之角nKm×nKm网格图 QGIS4.2.2
#该说明书仅作记录操作之用。需要代码的部分已使用*标注并另起一行

S1-加载非洲国家边界 attribute filtering then export
Natural Earth→Admin 0 Countries→下载ne_110m_admin_0_countries.shp(select Medium scale data,1:50m/cultural)→layer→add layer→add vector layer→选择上述文件
click layer→open attribute table→select a feature using expression(Epsilon)
*"ADMIN"IN('Sudan','South Sudan','Ethiopia','Eritrea',Djibouti','Somalia','Kenya')*
select→layer→export→save selected feature as...→保存horn of africa
outcome:a layer named'horn of afrcia'

S2-转换坐标系统 Reproject
（这一步的目的是将地图的经纬单位转换为米制单位，以便后面制作网格图）
click layer 'horn of africa'→export→save feature as→format→geopackage→set file name as 'horn of africa_projected'→select CRS(near the CRS)→search '102022'→click
outcome:a layer named 'horn of africa_projected'
注意：S1和S2可以合并操作，即在save select feature as之后就选择102022的CRS 

S3-创建网格
processing→toolbox→search'Creat grid'→grid type:Retangle→Grid extent:calculate from layer→select'horn of africa_projected'→horizontal spacing/vertical spacing:50km→run
outcome:a layer named'Grid';a gridded layer covering horn of africa
方法补充：裁剪网格的方式有三种，第一种是生成之后直接clip，优点是操作方便，缺点是小岛屿这类面积过小的区域被分割的很扭曲，遂放弃；第二种是生成网格后再在网格内生成centroid，再根据centroid生成网格单元（cell），优点是陆地能被完整保留且切割规整，缺点是有些小岛位于网格中心点之间，从grid到grid centroid之后中心点不落在小岛，根据中心点创建网格则小岛不被覆盖，并且因为这个原因，不规则的领土边缘也不被覆盖，遂放弃。该地图的目的是领土全覆盖，于是第三种是生成grid后select by location→grid→inersects→horn of africa_projected。只要有领土就保留cell，该方法能保证海岸、小岛全覆盖。

S4-筛选网格
processing-toolbox-select by location-select features from:'Grid'-where the features:intersect-by comparing to the features from:'horn of africa_projected'-layer of 'Grid'-save selected feature as...-保存'grid-horn of africa'
outcome:a layer named'grid-horn of africa_projected'

其他步骤
S5-显示国家名字
click layer'ne_50m_admin_o_countries'-properties-labels-select:single labels-value:ADMIN（这个要看具体的字段）-字体arial/大小10/buffer/1-2px

S6-调整透明度
click layer'grid-horn of africa_projected'→symbology→opacity，然后根据需要调整fill;stroke color/width（建议0.1mm）等，建议fill和stroke的调整分两个图层。

S7-取消选中部分的高亮
edit-select-deselect features from all layers

S8-处理地图当中政治不正确的内容
删除国家标签
layer→properties→lables→Epsilon(near the value)→expression→
*CASE
WHEN "ADMIN"='Somaliland'
THEN NULL
ELSE "ADMIN"
END*
删除国家边界
select features(select the target contries)-layer-toggle editing-Edit-edit geometry-merge selected features

