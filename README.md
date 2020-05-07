# GoogleMap_Spider
[iewoai]爬取谷歌地图的商业信息（requests版）

# 谷歌地图爬虫（requests版）
## 一、目标：搜素company，改坐标，获取全球所有公司信息
## 思路：从一点出发，选取该点最优Z（以整数调整），以该点的切片边长或1d表达式做距离，经纬等长移动度差，移动后的每一点重复该步骤。
### 注（纯个人理解）：
### 1. 同一经纬度，不同缩放倍数的公司数目不一样，很难选最优值
### 2. 当没有最优值，即该坐标附近无公司的情况，需设置默认度幅（经纬度增减幅度）
### 3. 移动时为前后左右移动，即固定经度或纬度，正负移动另一个
### 4. 固纬度动经度，度差受纬度和距离影响；固经度动纬度，度差仅受距离影响
### 5. 最优值选缩放倍数大于等于12的（纬度幅0.82739）小于等于18的（纬度幅0.00899）中公司数目中最多的倍数，以其1d（见url解析）做距离，上下左右移动。当12-18公司数都为0时，取默认度幅


## 二、瓦片地图原理参考资料：
### 1. https://segmentfault.com/a/1190000011276788
### 2. https://blog.csdn.net/mygisforum/article/details/8162751
### 3. https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection/
### 4. https://blog.csdn.net/mygisforum/article/details/13295223


## 三、url解析1
#### 例：url = 'https://www.google.com/maps/search/company/@43.8650268,-124.6372106,3.79z'
#### https://www.google.com/maps/search/company/@X,Y,Z
### 1. X 纬度，范围[-85.05112877980659, 85.05112877980659]
### 2. Y 经度，范围 [-180, 180]
### 3. Z 缩放倍数，范围[2, 21]
### 4. Z=2 切片正方形边长为20037508.3427892
## url解析2
#### 例：https://www.google.com/search?tbm=map&authuser=0&hl={1}&gl={2}&pb=!4m9!1m3!1d{3}!2d{4}!3d{5}!2m0!3m2!1i784!2i644!4f13.1!7i20{6}!10b1!12m8!1m1!18b1!2m3!5m1!6e2!20e3!10b1!16b1!19m4!2m3!1i360!2i120!4i8!20m57!2m2!1i203!2i100!3m2!2i4!5b1!6m6!1m2!1i86!2i86!1m2!1i408!2i240!7m42!1m3!1e1!2b0!3e3!1m3!1e2!2b1!3e2!1m3!1e2!2b0!3e3!1m3!1e3!2b0!3e3!1m3!1e8!2b0!3e3!1m3!1e3!2b1!3e2!1m3!1e9!2b1!3e2!1m3!1e10!2b0!3e3!1m3!1e10!2b1!3e2!1m3!1e10!2b0!3e4!2b1!4b1!9b0!22m6!1sg3qzXsG-JpeGoATHyYKQBw%3A1!2zMWk6MSx0OjExODg3LGU6MCxwOmczcXpYc0ctSnBlR29BVEh5WUtRQnc6MQ!7e81!12e3!17sg3qzXsG-JpeGoATHyYKQBw%3A110!18e15!24m46!1m12!13m6!2b1!3b1!4b1!6i1!8b1!9b1!18m4!3b1!4b1!5b1!6b1!2b1!5m5!2b1!3b1!5b1!6b1!7b1!10m1!8e3!14m1!3b1!17b1!20m2!1e3!1e6!24b1!25b1!26b1!30m1!2b1!36b1!43b1!52b1!55b1!56m2!1b1!3b1!65m5!3m4!1m3!1m2!1i224!2i298!26m4!2m3!1i80!2i92!4i8!30m28!1m6!1m2!1i0!2i0!2m2!1i458!2i644!1m6!1m2!1i734!2i0!2m2!1i784!2i644!1m6!1m2!1i0!2i0!2m2!1i784!2i20!1m6!1m2!1i0!2i624!2m2!1i784!2i644!34m13!2b1!3b1!4b1!6b1!8m3!1b1!3b1!4b1!9b1!12b1!14b1!20b1!23b1!37m1!1e81!42b1!47m0!49m1!3b1!50m4!2e2!3m2!1b1!3b0!65m0&q={7}&oq={8}&gs_l=maps.12...0.0.1.12357296.1.1.0.0.0.0.0.0..0.0....0...1ac..64.maps..1.0.0.0...3041.&tch=1&ech=1&psi=g3qzXsG-JpeGoATHyYKQBw.1588820611303.1
### 1. hl={1}为语言，常用zh-CN或en
### 2. g1={2}为当前所在国家地区缩写
### 3. !1d{3}为1d，与缩放倍数有关，相邻两整数缩放倍数之间1d成1/2关系，缩放倍数越高值越小，最大为94618532.08008283，猜测为当前所在地图切片边长或周长等边长正比关系
### 4. !2d{4}为2d，经度
### 5. !3d{5}为3d，纬度
### 6. {6}为搜素结果页数，格式为!8i+page，其中page默认为20的倍数，且page为0时（即第一页时），url中无!8i字段
### 7. q={7}&oq={8}，基本都是搜素词


## 四、经纬度和距离互转
### 1. 来源：https://blog.csdn.net/qq_37742059/article/details/101554565
