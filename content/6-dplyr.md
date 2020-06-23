---
layout: default
title: dplyr
parent: Outline
nav_order: 6
---

## dplyr

Desinty analysis and quadrat did not make use of the polygons representing each neighbourhood. Can we count the number of services
for a given neighbourhood?


R Code
{: .label .label-green }
```R
head(vancouver_boundaries$mapid)
```

Output
{: .label .label-yellow }
```R
[1] AR   CBD  FAIR GW   HS   MARP
Levels: AR CBD DS FAIR GW HS KC KERR KIL KITS MARP MP OAK RC RP SC SHAU STR SUN VF WE WPG
```


## Finding points within a polygon

R Code
{: .label .label-green }
```R
filtered <- vancouver_boundaries %>% filter(mapid == "MP")
st_intersects(filtered, schools)
lengths(st_intersects(filtered, schools))
```



## Finding points for all polygons

R Code
{: .label .label-green }
```R
lst_cnt <- c()
for (neighbourhood in vancouver_boundaries$mapid) {
  neighbourhood_polygon <- vancouver_boundaries %>% 
    filter(mapid == neighbourhood)

  cnt <- lengths(st_intersects(neighbourhood_polygon, schools))
  lst_cnt <- c(cnt, lst_cnt)
}
lst_cnt
```


R Code
{: .label .label-green }
```R
vancouver_boundaries$schools <- lst_cnt
vancouver_boundaries %>%
  pull(mapid, schools)
```

Output
{: .label .label-yellow }
```R
> vancouver_boundaries %>%
+   pull(mapid, schools)
   7    9   21    7    8   17   10    8   11    8    5    8    3    4   
  AR  CBD FAIR   GW   HS MARP   RP SHAU  STR   WE   DS KERR  KIL KITS   
```


R Code
{: .label .label-green }
```R
brks <- classIntervals(vancouver_boundaries$schools, n = 5, style = "quantile")
tm_shape(vancouver_boundaries) + 
  tm_polygons("schools", 
              breaks= brks$brks, title="Seervices",
              border.col = "white", palette = "GnBu") +
  tm_text("schools", just = "center", size = 0.8) +
  tm_legend(outside=TRUE)
```

Output
{: .label .label-yellow }
<img src="{{site.baseurl}}/content/fig/plot6.png">




## Exercise

- Count the number of community centres
{: .warn}



## Encapsulating everything in a function

R Code
{: .label .label-green }
```R
cnt_services <- function(point_data) {
  lst_cnt <- c()
  for (neighbourhood in vancouver_boundaries$mapid) {
    neighbourhood_polygon <- vancouver_boundaries %>% filter(mapid == neighbourhood)
    cnt <- lengths(st_intersects(neighbourhood_polygon, point_data))
    lst_cnt <- c(cnt, lst_cnt)
  } 
  
  return(lst_cnt)
}
```


R Code
{: .label .label-green }
```R
vancouver_boundaries$schools <- cnt_services(schools)
vancouver_boundaries$libraries <- cnt_services(libraries)
vancouver_boundaries$community_centres <- cnt_services(community_centres)


vancouver_boundaries$all_services <- vancouver_boundaries$schools + 
  vancouver_boundaries$libraries + 
  vancouver_boundaries$community_centres
```



R Code
{: .label .label-green }
```R
brks <- classIntervals(vancouver_boundaries$all_services, n = 5, style = "quantile")
tm_shape(vancouver_boundaries) + 
  tm_polygons("all_services", 
        breaks= brks$brks, title="Services",
        border.col = "white", palette = "GnBu") +
  tm_text("all_services", just = "center", size = 0.8) +
  tm_legend(outside=TRUE)
```



Output
{: .label .label-yellow }
<img src="{{site.baseurl}}/content/fig/plot7.png">



### Recap

- `filter` filter a dataset based on some condition
- `st_intersects` binds a polygon area to a set of points
- `classIntervals` divides data into buckets